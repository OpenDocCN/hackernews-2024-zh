<!--yml
category: 未分类
date: 2024-05-27 14:52:43
-->

# Clay Foundation Model — Clay Foundation Model

> 来源：[https://clay-foundation.github.io/model/](https://clay-foundation.github.io/model/)

## An open source AI model for Earth

Clay is a [foundation model](https://www.adalovelaceinstitute.org/resource/foundation-models-explainer/) of Earth. Foundation models trained on earth observation (EO) data can efficiently distill and synthesize vast amounts of environmental data, allowing them to generalize this knowledge to specific, downstream applications. This makes them versatile and powerful tools for nature and climate applications.

Clay’s model takes satellite imagery, along with information about location and time, as an input, and outputs embeddings, which are mathematical representations of a given area at a certain time on Earth’s surface. It uses a Vision Transformer architecture adapted to understand geospatial and temporal relations on Earth Observation data. The model is trained via Self-supervised learning (SSL) using a Masked Autoencoder (MAE) method.

The Clay model can be used in three main ways:

*   **Generate semantic embeddings for any location and time.** You can use embeddings for a variety of tasks, including to:

    *   *Find features:* Locate objects or features, such as surface mines, aquaculture, or concentrated animal feeding operations.

*   **Fine-tune the model for downstream tasks such as classification, regression, and generative tasks.** Fine-tuning the model takes advantage of its pre-training to more efficiently classify types, predict values, or detect change than from-scratch methods. Embeddings can also be used to do the following, which require fine-tuning:

    *   *Classify types or predict values of interest:* Identify the types or classes of a given feature, such as crop type or land cover, or predict values of variables of interest, such as above ground biomass or agricultural productivity.

    *   *Detect changes over time:* Find areas that have experienced changes such as deforestation, wildfires, destruction from human conflict, flooding, or urban development.

    *   This can be done by training a downstream model to take embeddings as input and output predicted classes/values. This could also include fine-tuning model weights to update the embeddings themselves.

*   **Use the model as a backbone for other models.**