# Distiset

Distilabel pipelines return a special type of dataset: [`Distiset`][distilabel.distiset.Distiset].

This class is a wrapper around `datasets.Dataset` which comes with some extra functionality to easily deal with the dataset pieces that a `Pipeline` can generate.

```python
from distilabel.distiset import Distiset
from datasets import Dataset

ds = Distiset(
    {
        "leaf_step_1": Dataset.from_dict({"instruction": [1, 2, 3]}),
        "leaf_step_2": Dataset.from_dict(
            {"instruction": [1, 2, 3, 4], "generation": [5, 6, 7, 8]}
        ),
    }
)
```

This object works like a python dictionary (the same approach followed by [`datasets.DatasetDict`](https://huggingface.co/docs/datasets/main/en/package_reference/main_classes#datasets.DatasetDict)), where each key corresponds to one of the `leaf_steps` from a `Pipeline`.

## Distiset methods

We can interact with the different pieces generated by the `Pipeline` and treat them as different [`configurations`](https://huggingface.co/docs/datasets-server/configs_and_splits#configurations). The `Distiset` contains just two methods:

### Train/Test split

Which easily does the train/test split partition of the dataset for the different configurations or subsets.

```python
>>> ds.train_test_split(train_size=0.9)
Distiset({
    leaf_step_1: DatasetDict({
        train: Dataset({
            features: ['instruction'],
            num_rows: 2
        })
        test: Dataset({
            features: ['instruction'],
            num_rows: 1
        })
    })
    leaf_step_2: DatasetDict({
        train: Dataset({
            features: ['instruction', 'generation'],
            num_rows: 3
        })
        test: Dataset({
            features: ['instruction', 'generation'],
            num_rows: 1
        })
    })
})
```

### Push to HuggingFace hub

Pushes the internal subsets to a huggingface repo, where each one of the subsets will be a different configuration, so it's easy to download them and continue working with any of the pieces.

```python
ds.push_to_hub(
    "my_org/dataset",
    commit_message="My first Distiset",
    private=False,
    token=os.getenv("HF_API_TOKEN"),
)
```

## Dataset card

Having this special type of dataset comes with an added advantage. When calling `Distiset.push_to_hub`:

```python
ds.push_to_hub("my_org/dataset", generate_card=True)
```

We will have an automatic dataset card (an example can be seen [here](https://huggingface.co/datasets/distilabel-internal-testing/deita)) with some handy information like reproducing the `Pipeline` with the `CLI`, or examples of the records from the different subsets.
