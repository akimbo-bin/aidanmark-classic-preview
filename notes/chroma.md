### Oscillator-Level Inverse Synthesis

This project is an audio-to-synth parameter inference system built for **Xfer Serum 2**. Given a short audio example, the system predicts a set of synthesizer parameters that recreate the core timbral character of the sound and packages the result as a real, loadable `.SerumPreset` file. The output is not just generated audio or a nearest-neighbour lookup result; it is a playable synth patch that can be opened, edited, automated, and used in a normal production workflow.

#### Overview

Modern synthesizers are extremely expressive, but that flexibility comes with a large parameter space and a lot of interdependence between controls. Serum exposes thousands of parameters, and even limiting the problem to oscillator-level controls still leaves a difficult inverse problem: many different parameter combinations can produce perceptually similar results, and the synthesizer itself is not differentiable in a way that makes direct gradient-based optimization practical.

The system focuses on the part of the problem that is both tractable and musically useful: predicting the oscillator configuration well enough to produce a strong starting patch. The initial scope includes wavetable selection, wavetable position, warp modes and amounts, unison settings, stereo width, blend, and oscillator level. Filters, effects, complex modulation, and secondary synthesis layers are intentionally excluded from the early phases so the model can learn the strongest timbral controls first.

#### End-to-end pipeline

The full pipeline has four main stages:

1. **Generate a labeled dataset**
2. **Convert audio into a stereo-aware representation**
3. **Predict synthesizer parameters with a multi-head neural network**
4. **Reconstruct the predictions as a valid Serum preset**

Each stage was built specifically for this project. There was no off-the-shelf dataset for Serum, no built-in parameter export pipeline, and no public API for direct preset generation.

#### Dataset generation

A large part of the work was building the dataset pipeline.

Because Serum 2 is a commercial VST and does not expose a public interface for randomizing parameters and exporting paired labels, the project uses **REAPER** and its **ReaScript API** as a host environment. Python scripts generate randomized oscillator configurations, apply them to Serum, render short audio clips offline, and export the exact parameter state alongside each render. That produces perfectly aligned audio–parameter pairs at scale.

The final dataset contains **193,283 labeled samples** across **18 wavetables** and multiple training phases. Each sample is a 2-second stereo render at **44.1 kHz**, giving roughly **107 hours** of generated training audio. Rather than trying to randomize everything at once, the data is built using a curriculum strategy: early phases focus on wavetable identity and position, later phases introduce warp modes, then unison and stereo controls, and then multi-oscillator combinations. This keeps the learning problem structured and avoids immediately exploding the parameter space.

#### Audio representation

The model does not operate on raw waveforms. Audio is converted into **LRMS mel spectrograms**, which preserve both spectral content and stereo information.

Instead of reducing the sound to mono, the preprocessing stack creates four channels:
- **Left**
- **Right**
- **Mid**
- **Side**

That is important for a synthesizer like Serum, where stereo width, unison detune, and blend have a clear spatial effect that would be weakened or lost in a mono representation. Spectrograms are computed with **128 mel bins**, **172 time frames**, a **2048-point FFT**, and a **512-sample hop length**. All features are precomputed and cached to disk as tensors so training is not bottlenecked by feature extraction.

#### Model architecture

The core model is a **7-layer convolutional neural network** implemented in **PyTorch**. It uses a shared encoder and multiple task-specific output heads.

The encoder takes the 4-channel LRMS spectrogram and progressively downsamples it into a compact **256-dimensional latent representation**. That latent vector is then passed to separate heads for different parameter types:
- **categorical heads** for wavetable selection, warp modes, and voice count
- **regression heads** for wavetable position, warp amount, detune, stereo width, blend, and level

This matters because the parameters are not all the same kind of prediction problem. Some are discrete class choices, others are bounded continuous controls, and they need different losses and output constraints.

Before full parameter training, the encoder is first pretrained as part of an **autoencoder**. The idea is to teach the model to build a general-purpose representation of timbre before asking it to predict exact synth controls. After pretraining, the decoder is discarded and the encoder weights are transferred into the final multi-head inference model.

#### Training strategy

The model is trained as a multi-task system with weighted losses per parameter. Categorical targets use cross-entropy losses, continuous targets use MAE or MSE depending on the parameter, and the training loop uses **phase masking** so each batch only trains the heads that are relevant for that curriculum phase.

That allows one architecture to grow with the dataset instead of training separate models for each subset of parameters. Early phases establish a strong representation of basic timbral structure, while later phases add more difficult controls without resetting what the model has already learned.

The optimizer is **AdamW** with cosine annealing, warm restarts, and early stopping. A separate lightweight CNN is also used to predict ADSR envelope parameters as a temporary measure while the main architecture remains focused on oscillator-level inference.

#### Preset reconstruction

Model predictions are converted back into real Serum presets.

This required reverse-engineering the mapping between Serum’s internal preset fields and the actual oscillator controls used by the synthesizer. The system starts from a clean preset template, injects the predicted values into the correct internal fields, resolves wavetable references, and repackages the result using Serum’s preset format so the patch loads directly inside the plugin.

That reconstruction step is what turns the project from a pure ML experiment into a working production tool. The output is a real preset file, not just a score or a visualization.

#### Results

The strongest results come from parameters with clear and stable spectral signatures.

- **Wavetable selection** reached **97.4% top-1 accuracy**
- **Warp mode 1** reached **78.9%**
- **Unison voice count** reached **74.0%** on balanced evaluation
- Continuous controls such as wavetable position, detune, stereo width, and oscillator level achieved usable error ranges, with oscillator level performing especially well

Listening tests are as important as the raw metrics here. Exact parameter recovery is often impossible because the mapping from sound to synth settings is inherently one-to-many, but the generated presets frequently capture the target’s harmonic shape, stereo character, and overall timbral identity closely enough to be musically useful with only minor adjustment.

#### What the project revealed

The most important result is not just that the system works. It is *where* it stops scaling cleanly.

Parameters with strong spectral signatures and balanced data learn well. Parameters with entanglement or class imbalance become much harder. Warp mode 2, for example, suffered from a dataset design issue in one phase and showed how quickly the task becomes fragile when the distinctions between classes are subtle. Unison voice count also showed the limits of learning under heavily imbalanced data.

That leads to the main engineering conclusion of the project: the biggest bottleneck is not the CNN architecture itself, but the cost of generating balanced data across combinations of parameters. Once filters, effects, envelopes, modulation routing, and multi-oscillator interactions are added, the number of meaningful combinations grows extremely quickly. At that point, the limiting factor becomes dataset design and coverage, not simply model size.

#### Outcome

The final result is a complete audio-to-preset system for Serum 2:
- automated dataset generation at scale
- stereo-aware audio preprocessing
- a multi-head CNN for parameter inference
- preset reconstruction into loadable Serum files

As a capstone project, it establishes that oscillator-level inverse synthesis is practical and usable, while also making the scaling limits of the parameter-prediction approach very clear. That combination is what makes it important: it produced a working system, and it also exposed the exact technical reasons for moving beyond parameter matching toward a different class of neural synthesis system.
