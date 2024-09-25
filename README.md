# Flux training

I made this repo to be able to share with you the details of how I fine-tuned the Flux model. The main steps were made following this [tutorial](https://medium.com/@geronimo7/how-to-train-a-flux1-lora-for-1-dfd1800afce5). 

## File structure

* **image-caption.ipynb**: code to automatically caption images to use for training.
* **train-flux.ipynb**: code to train the Flux model.
* **flux-lora-img-gen-results.ipynb**: code to run inference using the fine-tuned weights.

## Hardware requirements
In order to train you would need at least 24 GB VRAM, and currently there is support for single-GPU, so you should go for a SageMaker notebook running on a G5 or higher. For inference, currently you need a bit more, at least 28GB VRAM. I couldn't run inference on our ML-PT account, I ended up renting an instance on Vast.AI with 48GB VRAM. For storage, you would need at least 100GB of storage.

## Training steps

### Step 1: Clone repo and install dependencies
The tutorial I followed is based on the Ostris' [AI-Toolkit](https://github.com/ostris/ai-toolkit). You first begin by cloning that repo and installing it's dependencies. The tutorial uses /workspace as folder, you could choose whatever suits you. 

```
!cd /workspace 
!git clone https://github.com/ostris/ai-toolkit.git
!cd ai-toolkit && git submodule update --init --recursive && pip install -r requirements.txt
```

### Step 2: Upload images and generate captions
Then, you need to upload a folder with your images and captions. If you don't have captions, you can generate them automatically using the script in the `image-caption.ipynb` notebook. The code inside takes the folder with your images, generates a caption for every image and stores it on a .txt file with the same name. Keep in mind that you may need to adapt it to your specific folder structure.

### Step 3: Log into HuggingFace
Access to the FLUX1-dev model is gated, so you first have to accept their terms. Log into your Hugging Face account (or create one) and accept their terms: [FLUX1-dev repository](https://huggingface.co/black-forest-labs/FLUX.1-dev)

Next, generate a Hugging Face API token on your account and log in:
```
!huggingface-cli login --token hf_XXXXTOKENXXXX
```

### Step 4: Define training parameters

On the first cell of the `train-flux.ipynb` you would need to define:
* **INPUT_FOLDER**: where your images and captions are stored),
* **OUTPUT_FOLDER**: where to store results like samples and weights,
* **TRIGGER_WORD**: the name/word for the object or subject you are finetuning on. 

and a few other training parameters upon which you can play on, like after how many steps you would like to save the weights or produce sample images to measure progress. For your first time I would leave them as is, and adjust on subsequent training runs if you feel the need for it. 

### Step 5: Define job parameter dictionary

That's what the second cell of the notebook is doing. You can dig deeper into each parameter, my advice is that if you are training with limited VRAM, that you leave uncommented the `low_vram` parameter. If you have VRAM to spare, comment it so the training takes much less time.

### Step 6: Final step - run the training job

Finally, run the training job using the last cell of the notebook. The actual training time will vary depending on how many images you used, your GPU VRAM, how many steps you defined, etc. In my case, using 24GB VRAM and `low_vram` mode, it took around 4:30 hs to run 2250 training epochs.

## Inference

Finally, you can use the `flux-lora-img-gen-results.ipynb` notebook to use your fine-tuned model and generate cool images with it. There are several parameters to consider when running the inference:

* prompt and negative prompt: the actual textual description of the image you want to generate. Apparently Flux relies on plain text prompts, different to Stable diffusion which had a bunch of flags and parameters. For inspiration you could visit [PromptHero](https://prompthero.com/flux-prompts?__cf_chl_tk=nKmeQBc9IU6dIH9o44wP3ak3HplrZ71Rfq_jM1gC8k4-1727291842-0.0.1.1-7956), even though I found it to be quite biased towards suggestive images 😒. Also, I left a couple interesting prompts on the inference notebook.
* num_inference_steps: how many inference passes the model does before returning the image. You would have to find a number that gives you the best number, I found that between 20 and 40 I had the best results.
* width
* height

## Extra info

Bojan Jakimovski shared with me this other repository for fine-tuning Flux that looks even easier [FluxGym](https://github.com/cocktailpeanut/fluxgym). I believe is from the guy that made the AI-Browser [Pinokio](https://pinokio.computer/). If anyone tries it out, please let me know and we could add your experience to this repo.
