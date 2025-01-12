# Klicka Action Model PoC
This repository contains a proof-of-concept of Klicka AIs first generation Transformer model called "UI-act" for interacting with a computer using the graphical user interface. 
The model shares the same interface as a human operator, i.e. screen input and low-level mouse/keyboard output.
The motivation for this is to allow for seamless integration into human-computer workflows, where the model can naturally be trained using expert human demonstrations.

https://github.com/TobiasNorlund/UI-Act/assets/2678217/5683b08f-b9b0-401d-9984-e7fdb67e064e

## How it works

UI-Act is an encoder-decoder Transformer model, similar to a language model like T5.
The core difference is that the _decoder_ takes a screenshot as input, and predicts a low-level action such as a mouse click on a specific pixel location.
The action is also conditioned on text via cross-attention to the _encoder_, enabling instructive prompting.
The input screenshot is encoded into a (single) embedding using a convolutional neural network before being fed into the decoder.

![UI-Act model](/data/UI-Act-model.png)

Once the predicted action has been executed, a new screenshot is taken, encoded, and fed into the next position of the decoder.
This allows the action to also be conditioned on previous states, via causal self-attention in the decoder.

The actions are predicted using two linear heads based on the output hidden states from the decoder.
The first head classifies the _action_type_, including EOS. In the demo, only left clicks are valid.
In case of a click prediction, the second head classifies the (flattened) pixel location. 
To limit the number of locations, we classify onto a 4x4 pixel grid.  

For more details on the model architecture, see [ui_act/model.py](ui_act/model.py).


## Demo: Add and subtract using GNOME Calculator in Ubuntu

In the above demo, the model has been trained to compute simple (up to 2-digit) arithmetic expressions using an open and visible GNOME Calculator window.

The model is trained end-to-end from scratch on demonstrations, i.e. behavioral cloning, and learns implicitly to perform OCR to detect the calculator buttons.
When trained on demonstrations adding or subtracting numbers between 0-50, it also generalizes to numbers between 50-99.
Due to the translational invariant CNN representations, the window can be put anywhere on the screen, and in any size.
However, the model is very overfit to the visual environment it is trained on, and is easily confused by a new desktop background or resolution.
Having other windows open also puts it out of distribution.

This demo model is very small, with a total of only 1.8M parameters. This makes it lightweight to run, even on a laptop CPU.

**Try on your own:**

1. [Download](https://storage.googleapis.com/ui-act/UI-Act-VM.ova) this VirtualBox VM (user: ui-act, pw: ui-act)
2. [Import](https://www.alphr.com/ova-virtualbox/) it to VirtualBox
3. Start it, you'll be automatically signed in
4. If not set already, set the desktop resolution to 1920x1080 which is what the model was trained using
5. Open a Calculator window from the Ubuntu taskbar
6. Open a terminal and run:
```bash
# Navigate to the already cloned repo
cd ~/UI-Act

# Run start script to build/run provided docker container
./start.sh

# In docker, run demo script
python -m ui_act.tasks.gnome_calculator.run

# When prompted, enter an arithmetic expression (it's been trained to compute [0-50][+-][0-50])
```
