# MEI: Multi-partition Embedding Interaction

This is a pure Python code implementing the MEI knowledge graph embedding method in the paper [Multi-Partition Embedding Interaction with Block Term Format for Knowledge Graph Completion](https://arxiv.org/abs/2006.16365) (ECAI 2020). MEI unifies several recent state-of-the-art KGE models and provides an optimal efficiency-expressiveness solution. The code is optimized for high performance in PyTorch, demonstrates several important KGE techniques, and features some recent state-of-the-art models including W2V, DistMult, CP, SimplE, ComplEx, RotatE, Quaternion, and MEI.

Knowledge graph embedding methods (KGE) aim to learn low-dimensional vector representations of entities and relations in knowledge graphs. The models take input in the format of triples (h, t, r) denoting head entity, tail entity, and relation, respectively, and output their embedding vectors as well as solving the link prediction task. For more information, please see our paper.

![MEI architecture](./MEI_architecture.png)
<p align="center">
  <i><b>Figure 1:</b> Architecture of MEI with block term format in three different views for the local interaction: Tucker format, parameterized bilinear format, and neural network format.</i>
</p>

## Installation
- Clone the repository to your local machine: `git clone https://github.com/tranhungnghiep/MEI-KGE/`
- Go to the repository directory: `cd MEI-KGE/`
- Install required packages, you may install in a separate environment: `pip install -r requirements.txt`

## How to run
Go to the source directory (`cd src/`) and run the following commands.

On WN18RR, MEI 3x100:
```shell script
python main.py --seed 7 --config_id "rep" --gpu 0 --model MEI --in_path ../datasets/wn18rr/ --out_path ../result/ --check_period 5 --K 3 --Ce 100 --Cr 100 --reuse_array torch1pin --sampling kvsall --loss_mode softmax-cross-entropy --batch_size 1024 --max_epoch 1000 --opt_method adam --amsgrad 0 --lr 3e-3 --lr_scheduler exp --lr_decay 0.99775 --lambda_ent 0.0 --lambda_rel 0.0 --lambda_params 0.0 --label_smooth 0.0 --constraint "" --to_constrain "" --lambda_rowrelnorm 0.0 --droprate_w 0.0 --droprate_r 0.0 --droprate_mr 0.0 --droprate_h 0.71 --droprate_mrh 0.67 --droprate_t 0.0 --norm bn --n_w 0 --n_r 0 --n_mr 0 --n_h 1 --n_mrh 1 --n_t 0 --n_sepK 1
```
On FB15K-237, MEI 3x100:
```shell script
python main.py --seed 7 --config_id "rep" --gpu 0 --model MEI --in_path ../datasets/fb15k-237/ --out_path ../result/ --K 3 --Ce 100 --Cr 100 --reuse_array torch1pin --sampling 1vsall --loss_mode softmax-cross-entropy --batch_size 1024 --max_epoch 1000 --opt_method adam --amsgrad 1 --lr 3e-3 --lr_scheduler exp --lr_decay 0.99775 --lambda_ent 0.0 --lambda_rel 0.0 --lambda_params 0.0 --label_smooth 0.0 --constraint "" --to_constrain "" --lambda_rowrelnorm 0.0 --droprate_w 0.0 --droprate_r 0.0 --droprate_mr 0.0 --droprate_h 0.66 --droprate_mrh 0.67 --droprate_t 0.0 --norm bn --n_w 0 --n_r 0 --n_mr 0 --n_h 1 --n_mrh 1 --n_t 0 --n_sepK 0
```
On YAGO3-10, MEI 5x100:
```shell script
python main.py --seed 7 --config_id "rep" --gpu 0 --model MEI --in_path ../datasets/YAGO3-10/ --out_path ../result/ --K 5 --Ce 100 --Cr 100 --reuse_array torch1gpu --sampling 1vsall --loss_mode softmax-cross-entropy --batch_size 1024 --max_epoch 1000 --opt_method adam --amsgrad 1 --lr 3e-3 --lr_scheduler exp --lr_decay 0.995 --lambda_ent 0.0 --lambda_rel 0.0 --lambda_params 0.0 --label_smooth 0.0 --constraint "" --to_constrain "" --lambda_rowrelnorm 0.0 --droprate_w 0.0 --droprate_r 0.0 --droprate_mr 0.0 --droprate_h 0.1 --droprate_mrh 0.15 --droprate_t 0.0 --norm bn --n_w 0 --n_r 0 --n_mr 0 --n_h 1 --n_mrh 1 --n_t 0 --n_sepK 0
```

## Results
The above hyperparameters were tuned for MRR on the validation sets, higher MRR is better.

MEI achieves state-of-the-art results using quite small number of parameters.

| WN18RR              | MR | MRR | H@1 | H@3 | H@10 |
| ------------------- | - | - | - | - | - |
| MEI 3x100, valid set| 3102.417 | 0.481 | 0.448 | 0.492 | 0.544 |
| MEI 3x100, test set | 3291.269 | 0.483 | 0.447 | 0.497 | 0.553 |

| FB15K-237           | MR | MRR | H@1 | H@3 | H@10 |
| ------------------- | - | - | - | - | - |
| MEI 3x100, valid set| 134.995 | 0.370 | 0.278 | 0.404 | 0.554 |
| MEI 3x100, test set | 141.244 | 0.364 | 0.270 | 0.398 | 0.550 |

| YAGO3-10            | MR | MRR | H@1 | H@3 | H@10 |
| ------------------- | - | - | - | - | - |
| MEI 5x100, valid set| 834.918 | 0.579 | 0.508 | 0.619 | 0.706 |
| MEI 5x100, test set | 756.062 | 0.578 | 0.505 | 0.622 | 0.710 |

## How to implement your own KGE model
It is easy to implement new KGE model by extending the MEI class and only rewriting the score function. Our framework provides the loss functions, regularizations, constraints, experiments, evaluations, data sampling, etc. for training and evaluating quickly. For example:

```python
class YourKGE(MEI):
    def __init__(self, data, config):
        super().__init__(data, config)

    def compute_score(self, h: torch.Tensor, r: torch.Tensor) -> torch.Tensor:
        """
        Main logic: compute the score.
        Input: tensor in batch: (batch,) of indices (1-hot vector)
        :return: tensor in batch: (batch, num_ent) scores
        """
        # Example of trilinear dot product score. Feel free to change to your score function.
        score = (self.ent_embs[h] * self.rel_embs[r]) @ self.ent_embs.permute(1, 0)
        return score
```

## How to cite
If you found this code or our work useful, please cite us.
- *Hung Nghiep Tran and Atsuhiro Takasu. [Multi-Partition Embedding Interaction with Block Term Format for Knowledge Graph Completion](https://arxiv.org/abs/2006.16365). In Proceedings of the European Conference on Artificial Intelligence (ECAI), 2020.*  
  ```
  @inproceedings{tran_multipartitionembeddinginteraction_2020,
    title = {Multi-{Partition} {Embedding} {Interaction} with {Block} {Term} {Format} for {Knowledge} {Graph} {Completion}},
    booktitle = {Proceedings of the {European} {Conference} on {Artificial} {Intelligence}},
    author = {Tran, Hung Nghiep and Takasu, Atsuhiro},
    year = {2020},
    pages = {833--840},
    url = {https://arxiv.org/abs/2006.16365},
  }
  ```
- *Hung Nghiep Tran and Atsuhiro Takasu. [MEIM: Multi-partition Embedding Interaction Beyond Block Term Format for Efficient and Expressive Link Prediction](https://arxiv.org/abs/2209.15597). In Proceedings of the International Joint Conference on Artificial Intelligence (IJCAI), 2022.*  
  ```
  @inproceedings{tran_meimmultipartitionembedding_2022,
    title = {{MEIM}: {Multi}-partition {Embedding} {Interaction} {Beyond} {Block} {Term} {Format} for {Efficient} and {Expressive} {Link} {Prediction}},
    booktitle = {Proceedings of the {Thirty}-{First} {International} {Joint} {Conference} on {Artificial} {Intelligence}},
    author = {Tran, Hung Nghiep and Takasu, Atsuhiro},
    year = {2022},
    pages = {2262--2269},
    url = {https://arxiv.org/abs/2209.15597},
  }
  ```

## See also
- AnalyzeKGE, preliminary experiments and analysis: https://github.com/tranhungnghiep/AnalyzeKGE
- KG20C, a scholarly knowledge graph benchmark dataset: https://github.com/tranhungnghiep/KG20C
