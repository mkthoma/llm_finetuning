# Finetuning of Open Source LLM Models
This repo contains the code for finetuning open source LLM models, specifically the [Microsoft Phi2](https://huggingface.co/microsoft/phi-2) model that was recently released. It has 2.78 billion parameters. 

Phi-2 is a Transformer with 2.7 billion parameters. It was trained using the same data sources as [Phi-1.5](https://huggingface.co/microsoft/phi-1_5), augmented with a new data source that consists of various NLP synthetic texts and filtered websites (for safety and educational value). When assessed against benchmarks testing common sense, language understanding, and logical reasoning, Phi-2 showcased a nearly state-of-the-art performance among models with less than 13 billion parameters.

The model hasn't been fine-tuned through reinforcement learning from human feedback. The intention behind crafting this open-source model is to provide the research community with a non-restricted small model to explore vital safety challenges, such as reducing toxicity, understanding societal biases, enhancing controllability, and more.

The finetuning code is taken from [this repo](https://github.com/mshumer/gpt-llm-trainer/blob/main/README.md) with a few minor changes for Phi2. The dataset used for finetuning was taken from the [OpenAssistant Conversations Dataset](https://huggingface.co/datasets/OpenAssistant/oasst1?row=0) (OASST1).

The Hugging Face live implementation can be found [here](https://huggingface.co/spaces/mkthoma/Phi2_Chatbot). Keep in mind that it is hosted on a CPU and inference can take long waiting time.

## Preparing finetuning data
If you take a look at the dataet used for finetuning, you can see that each prompt and the following conversations (prompt and response) are stored in a tree like structure. 

```
{
  "message_tree_id": "14fbb664-a620-45ce-bee4-7c519b16a793",
  "tree_state": "ready_for_export",
  "prompt": {
    "message_id": "14fbb664-a620-45ce-bee4-7c519b16a793",
    "text": "Why can't we divide by 0? (..)",
    "role": "prompter",
    "lang": "en",
    "replies": [
      {
        "message_id": "894d30b6-56b4-4605-a504-89dd15d4d1c8",
        "text": "The reason we cannot divide by zero is because (..)",
        "role": "assistant",
        "lang": "en",
        "replies": [
          // ...
        ]
      },
      {
        "message_id": "84d0913b-0fd9-4508-8ef5-205626a7039d",
        "text": "The reason that the result of a division by zero is (..)",
        "role": "assistant",
        "lang": "en",
        "replies": [
          {
            "message_id": "3352725e-f424-4e3b-a627-b6db831bdbaa",
            "text": "Math is confusing. Like those weird Irrational (..)",
            "role": "prompter",
            "lang": "en",
            "replies": [
              {
                "message_id": "f46207ca-3149-46e9-a466-9163d4ce499c",
                "text": "Irrational numbers are simply numbers (..)",
                "role": "assistant",
                "lang": "en",
                "replies": []
              },
              // ...
            ]
          }
        ]
      }
    ]
  }
}
```

In order to parse this into a dataset format with each prompt and response I have used the notebooks for [train](https://github.com/mkthoma/llm_finetuning/blob/main/Data%20Prep/OpenAssistant%20Finetuning%20Training%20Data%20Prep.ipynb) and [validation](https://github.com/mkthoma/llm_finetuning/blob/main/Data%20Prep/OpenAssistant%20Finetuning%20Validation%20Data%20Prep%20(1).ipynb) dataset respectively. The parsed dataset is used for finetuning the model.

## Finetuning the model
The different parameters for the model are defined in the hyperparameters section of the notebook. The dataset is read, and the instructions for the user and system prompt are added, and then passed to the trainer for the model to be finetuned. 

The finetuned model is then saved, merged with the base model of Phi2 and then saved to the Google Drive storage. We are then able to load the model and then make inferences based on the prompts provided and the maximum token length. 

I have used a A100 instance for 1 epoch of finetuning which took about 4 hours to finish.
