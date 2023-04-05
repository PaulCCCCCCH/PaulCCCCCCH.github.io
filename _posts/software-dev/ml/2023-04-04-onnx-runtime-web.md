---
title: Recommendation System with Perfect Privacy - ONNX Runtime Web
date: 2021-04-04 21:17:00 -0500
author: Chonghan Chen
categories: [Software Development, Machine Learning]
tags: [notes, experience, tutorials, software engineering, machine learning]
math: true
---

# Recommendation System with Perfect Privacy - ONNX Runtime Web

The source code of the demo project is available [here](https://github.com/PaulCCCCCCH/onnx-recommender-demo).

## Prelude

Sometime ago, a friend of mine drove me to a party with his new car. During the ride, we discussed why he bought the car, how the automobile market was, etc. The next day, when I logged into social media, I was flooded with car advertisements. I was again impressed by the development of the eavesdropping network. I'm pretty sure we all have experienced this to some degree. Privacy violation has been a problem that is so serious and omniscient that most people would just accept it as if they are doomed, or act helplessly against the monopoly of big tech companies. From my point of view, the status quo can result from the companies' deliberate ignorance of user's privacy in exchange for profit, or the failure to implement the technology that can protect users' privacy while still being able to optimize for users' personal experience. With the assumption that the second one is true, we are going to explore some technicalities of making privacy-preserving AI recommendations in this post.



## The Problem and the Solution

For the rest of the post, let's use the movie streaming scenario as a concrete example. Suppose there is a company that provides movie streaming service (similar to Netflix). To better personalize users' experience, they would like to implement a system that recommends movies to users on their homepage. To achieve this, they have to collect data from users. However, users might be very worried about this part: can the company guarantee that the data they collect is only used for making movie recommendations? Could it be the case that after watching a movie with a lot of driving scenes, you suddenly start getting advertisements on their social media accounts? No one knows. Once your data is handed over to the company, you simply lose control, despite all the fancy promises they could make. If you were the CTO of the company, and given that you did want to protect users' privacy, how could you implement the system to eliminate users' worries? You have to run your model with the user's data no matter what. Well, the answer is simple: instead of having users handing in their data to you, you hand in your model to the users. In our case, you can have users run your recommendation model on their browser with the data that they have locally. This is easily done with ONNX.



## ONNX and ONNX Web Runtime

ONNX stands for Open Neural Network Exchange. [A previous post](https://sanglee325.github.io/ml/onnx/#) has covered what ONNX is, its strengths and weaknesses, with an example usage. To summarize, ONNX is a platform agnostic format for saving, loading and serving a model. It allows you to, let's say, train a model with PyTorch, save it in ONNX format, then load and use it in Tensorflow, Keras, C++, Javascript, or any languages and frameworks that has ONNX interface. If we want to run an ONNX model on users' browsers, we could use [ONNX Web Runtime](https://onnxruntime.ai/docs/tutorials/web/). It provides a set of Javascript APIs that allows you to easily load a model and use it to make predictions. We will demonstrate how it can be used within a demo project.



## Develop a Model

You may develop a model with your favorite language and framework. Be it PyTorch or Tensorflow, you are very likely to find ONNX support for that. In our demo project, we will be using `scikit-learn` and Python. We will create a simple linear regression model. Given a user and a movie, we will use this model to predict the rating the user would give to the movie. In this way, we can recommend movies that we think users will give the highest ratings to.

A simple model training pipeline is implemented in `models/get_regression_onnx_model.py`. The pipeline will initialize a model, load a fake training dataset, train the model, then save the model weight in ONNX format. You can run the pipeline with.

```
cd models
python get_regression_onnx_model.py
```

When it's done, you will see a model weight file in`./models/linear_reg_recommender.onnx`. The model will take 16 input features and predict a rating as a float number. Now that this part is ready, let's try serving it to the users from their browsers. For this, we will set up a fake website which calls the model to make predictions.



## Build the Website 

We will create a `react` frontend project with `create-react-app` as follows:

```
npx create-react-app onnx-recommender --template typescript
```

Then, install ONNX Web Runtime dependencies:

```
npm install --save onnxruntime-web ndarray
```

To make development easier, we also install a popular UI library `antd`

```
npm install --save antd
```

We will create a fake `Login` which handles user login logic (any password will be accepted) and a `Recommender` component that predicts the current user's rating given a movie. The directory structure looks like

```
src
└───Components
    └────Login
    	└───index.tsx
    └────Recommender
    	└───index.tsx
```

It's worth mentioning that, in `Recommender`, we use the following variables to mimic the data needed to make the prediction. In a real production scenario, `localUserFeatureStore` will the (encrypted) data of the users that have logged in from the device. It will be stored on the device and will never be given to the company. This ensures perfect safety of user's private information. On the other hand, `remoteMovieFeatureStore` is stored in the company's database, and can be fetched by the user at any time.

```javascript
const localUserFeatureStore: { [key: string]: UserFeature } = {
    "guest": {
        age: 35,
        gender: [0.3333333, 0.3333333, 0.3333333],
        occupation: [0.25, 0.25, 0.25, 0.25],
    },
    "paulcccccch": {
        age: 24,
        gender: [0.9, 0.05, 0.05],
        occupation: [0.2, 0.5, 0.3, 0.4],
    },
    "nobita": {
        age: 10,
        gender: [0.5, 0.25, 0.25],
        occupation: [0.1, 0.2, 0.7, 0.3],
    }
}

const remoteMovieFeatureStore: { [key: string]: MovieFeature } = {
    "Movie X": {
        genre: [0.12, 0.23, 0.34, 0.45],
        actors: [0.45, 0.34, 0.23, 0.12]
    }
}
```

We can follow [this document](https://onnxruntime.ai/docs/tutorials/web/) or [this example](https://github.com/microsoft/onnxruntime-web-demo) to prepare the input and retrieve the model's inference result. After you login successfully, you will be able to see a rating as below:

![prediction-page](/assets/img/software-dev/prediction-page.png)


In real-world applications, the logic could be much more complicated than this. What we have done so far serves as a minimal workable component that can be extended to fit in different applications.



## Discussion

Although serving models at the frontend allows companies to create a recommendation system that perfectly protects user's privacy, it has quite a few drawbacks compared to traditional cloud-centric recommendation:

- Not all models support ONNX conversion out-of-box. For example, only the listed models of `scikit-learn` are supported (see [here](https://onnx.ai/sklearn-onnx/supported.html)). If there is no ONNX support out there, you may have to create your own serializer for your model, which can be time consuming.
- Models are now static files that can be cached at multiple places. When you try to update the model, you will have to invalidate the cache, which adds additional complexity to your pipeline.
- Without users sending data to the server, it will be more difficult for the company to continuously improve their model.
- Users will have to download the models from time to time when visiting the website, which can result in a lot of network traffic and disk usage.

However, if a company really wants to protect user's privacy, these issues are by no means unsolvable. For example, the models can be designed in a way that only a small portion of the weights need to be updated frequently. Also, the company could let users send back model weight updates, which allows the company to keep improving their model with users data while preserving users' privacy.



## References

- [A website built with React and legacy ONNX.js](https://heartbeat.comet.ml/building-an-image-recognition-app-using-onnx-js-c7147f4f291b)
- [Official ONNX Runtime Web demo](https://github.com/microsoft/onnxruntime-web-demo)
- [ONNX Model ZOO](https://github.com/onnx/models)
- [Converting `scikit-learn` model to ONNX](https://onnx.ai/sklearn-onnx/)
- [ONNX Web Runtime doc](https://onnxruntime.ai/docs/tutorials/web/)

