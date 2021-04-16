# WIP: femb - Simple Face Embedding Training Library

```python
from femb.backbones import build_backbone
from femb.headers import ArcFaceHeader
from femb import FaceEmbeddingModel

# build the backbone embedding network
backbone = build_backbone(backbone="iresnet18", embed_dim=embed_dim)

# create one of the face recognition headers
# header = ArcFaceHeader(in_features=embed_dim, out_features=train_n_classes)
header = MagFaceHeader(in_features=embed_dim, out_features=train_n_classes)

# create the ce loss
loss = torch.nn.CrossEntropyLoss()

# create the face recognition model wrapper
face_model = FaceEmbeddingModel(backbone=backbone, header=header, loss=loss)
```

#### Basic Framework:
+ **Backbone**: The actual embedding network that we want to train. It takes some kind of input and produces a feature representation (embedding) of a certain dimensionality.
+ **Header**: A training-only extension to the backbone network that is used to predict the identity class logits for the loss function. This is the main part where the implemented methods (SphereFace, CosFace, ...) differ.
+ **Loss**: The loss function that is used to judge how good the (manipulated) logits match the one-hot encoded identity target. Usually, this is the cross-entropy loss.

```python
from femb.evaluation import VerificationEvaluator

# create the verification evaluator
evaluator = VerificationEvaluator(similarity='cos')

# specify the optimizer (and a scheduler)
optimizer = torch.optim.SGD(params=face_model.params, lr=1e-2, momentum=0.9, weight_decay=5e-4)
scheduler = torch.optim.lr_scheduler.MultiStepLR(optimizer=optimizer, milestones=[8000, 10000, 160000], gamma=0.1)

# fit the face embedding model to the dataset
face_model.fit(
    train_dataset=train_dataset,        # specify the training set
    batch_size=32,                      # batch size for training and evaluation
    device='cuda',                      # torch device, i.e. 'cpu' or 'cuda'
    optimizer=optimizer,                # torch optimizer
    lr_epoch_scheduler=None,            # scheduler based on epochs
    lr_global_step_scheduler=scheduler, # scheduler based on global steps
    evaluator=evaluator,                # evaluator module
    val_dataset=val_dataset,            # specify the validation set
    evaluation_steps=10,                # number of steps between evaluations
    max_training_steps=20000,           # maximum number of (global) training steps (if zero then max_epochs count is used for stopping)
    max_epochs=0,                       # maximum number of epochs (if zero then max_training_steps is used for stopping)
    tensorboard=True                    # specify whether or not tensorboard shall be used for embedding projections and metric monitoring
    )
```


#### Implemented Losses
+ **SoftMax Loss**: (LinearHeader: <img src="https://render.githubusercontent.com/render/math?math=\cos(\theta)">)
+ **SphereFace Loss**: [Paper](https://arxiv.org/abs/1704.08063) [Code](https://github.com/wy1iu/sphereface) (SphereFaceHeader: <img src="https://render.githubusercontent.com/render/math?math=\cos(m \cdot \theta)">)
+ **CosFace Loss**: [Paper](https://arxiv.org/abs/1801.09414) Code (CosFaceHeader: <img src="https://render.githubusercontent.com/render/math?math=\cos(\theta) - m">)
+ **ArcFace Loss**: [Paper](https://arxiv.org/abs/1801.07698) [Code](https://github.com/deepinsight/insightface/tree/master/recognition/arcface_torch) (ArcFaceHeader: <img src="https://render.githubusercontent.com/render/math?math=\cos(\theta %2B m)">)
+ **MagFace Loss**: [Paper](https://arxiv.org/abs/2103.06627) [Code](https://github.com/IrvingMeng/MagFace) (MagFaceHeader: <img src="https://render.githubusercontent.com/render/math?math=\cos(\theta %2B f_m(x))">)

#### TODOS
- [x] Add links to papers
- [x] Add inference methods to ```model.py ```
- [ ] Add comments and documentation
- [ ] Refactor code
- [ ] Test implementation 
