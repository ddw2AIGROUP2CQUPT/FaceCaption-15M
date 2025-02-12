# FLIP (Facial Language Image Pretrain)

**[24/07/05] 🤗FacaCaption-15M [OpenFace-CQUPT/FaceCaption-15M](https://huggingface.co/datasets/OpenFace-CQUPT/FaceCaption-15M)**

Based on FaceCaption-15M, we trained a multimodal representation model (FLIP), similar in concept to CLIP, designed for aligning facial images with semantics. FLIP contains the following components: (1) Image Encoder: Composed of a visual transformer, this component processes the image. (2) Text Encoder: When handling text input alone, this encoder follows the standard BERT module and uses the [CLS] token to summarize the entire sentence. In the case of multimodal input, a cross-attention layer is introduced between the self-attention layer and the feedforward network of the text encoder to fuse visual information (Image-grounded Text Encoder). To adapt to specific tasks, an [ENC] token is added to the text input, serving as the multimodal representation for the image-text pair.  

Checkpoints of FLIP is at：(https://huggingface.co/OpenFace-CQUPT/Facial-language-image-pretraining-model/)

**Overview of FLIP architecture.**

![image-20240318101027127](https://img.yutangli.net/img/202403181010116.png)

 **(a). Same color represents shared parameters. “12x” stands for 12-layer transformer modules. (b), (c) and (d) FLIP-based model are applied to the tasks of text-image retrieval, facial attributes prediction and sketch less facial image retrieval, respectively.**

# 1. Performance.

### Task1: Text-Image Retrieval

**Comparison with other classical pretrained models. All pretrained model backbones are frozen, with only the linear layer being fine-tuned. † represents the model pretrained on the LAION-Face [86] dataset; * represents the model pretrained on the FaceCaption dataset constructed without using LLM text generation.**

![](https://img.yutangli.net/img/202403181015142.png)

### Task2: Facial Attributes Prediction

**Comparison with other classical models. † represents the model pre-trained on the original LAION-Face dataset.**

![image-20240318101126897](https://img.yutangli.net/img/202403181011115.png)

### Task3: Sketch Less Facial Image Retrieval

**Comparative results with different baseline methods. † represents the model pre-trained on the LAION-Face dataset.**

![image-20240318101633671](https://img.yutangli.net/img/202403181016876.png)

**Performance of early retrieval in SLFIR problem. Instead of showing the complete sketch, we visualized it using the percentage of sketch. A higher value indicates a better early retrieval performance.**

![image-20240318101704679](https://img.yutangli.net/img/202403181017013.png)



# 2. Pipeline of our FaceCaption-15M construction process.

![image/png](https://cdn-uploads.huggingface.co/production/uploads/663f06e01cd68975883a353e/TCvUu0PlfC26BDbiKM5My.png)

## 2.1 Facial Images Collection
**Image Collection.** Specifically, we accessed the LAION-Face dataset, which contains over 50M image-text pairs that obtained through web crawling, as our source of raw data. LAION-Face is of a considerable scale, and its image distribution closely resembles real-world distributions. Moreover, using such a dataset as our raw data source offers significant cost savings compared to manual data collection. There were limitations stemming from link expiration and network issues, as we could only access about 75% of the LAION-Face.   
**Face Segmentation.** For original LAION-Face dataset, we segment the image of the facial regions. First, we selected all images with faces from LAION-Face using RetinaFace model, which resulted in approximately 37M images. To obtain a high-quality facial image dataset while avoiding noise interference, we conducted cropping, alignment, and filtering of the facial images based on facial region detection boxes. Specifically, we retained only those facial regions with resolutions exceeding 128 × 128 pixels and confidence scores higher than 0.98, resulting in approximately 23M images. Importantly, to maintain image quality, we did not uniformly scale the images to the same size, resulting in varying resolutions among the collected images.

## 2.2 Facial Attributes Annotation
Attributes play a pivotal role in generating the description text for facial image, thereby determining the correlation between the image and text. We designed 40 appearance attributes for facial features. Given the considerations of annotating a vast amount of data, we selected an automatic annotation method. In terms of efficiency and accuracy, we employed an open-source algorithm for predicting image attributes. To enhance the reliability of annotations, we retained only the labels predicted by the model with a probability exceeding 0.85. Furthermore, to generate more accurate natural language text, we retained samples with more than five valid predicted labels. Finally, we reduced the dataset size to 15M.

## 2.3 Facial Caption Generation: Raw Text Generation and Rewriting
Since, image-text pairs in LAION-Face dataset were obtained through subtitle crawling, and the text showed a weak correlation with the accompanying image. Our aim is to generate the caption of facial images. The manual annotation, while accurate, is time-consuming and labor-intensive, making it unsuitable for constructing large-scale datasets. However, automatic methods often offer efficiency and scalability. Nevertheless, the diversity, complexity, and naturalness of description sentences generated by traditional automatic text generation methods are limited by grammatical templates. With the development of LLM, text generated by these models is endowed with high diversity nd naturalness. Here, we propose a text generation strategy that combines grammatical templates with LLM. Specifically, (1) we first input the attribute annotations generated by Section 3.2 into the designed grammatical template to generate the raw text, and then (2) we input the raw text into the LLM to generate natural, diverse, and accurate text descriptions.  
To ensure the generation of high-quality description text using LLM, the quality of the raw text generated by the grammatical template is paramount. Here, we adopted the probabilistic context-free grammars (PCFG) algorithm to generate the raw text as multiple short sentences, each constructed using different attributes. The performance of the LLM model itself affects the quality of the generated caption. We conducted research on existing open-source LLMs and finally selected the Qwen-7B-Chat model.


## 2.4 Statistical Analysis for FaceCaption-15M Dataset
**Comparisons with other popular facial image datasets.** Symbol “#” indicates the number of samples (images or image-text pairs). Abbreviations “mRes”, “Ann”, and “mWords” denote average resolution of all images, the number of annotations, and average words of all text, respectively. Abbreviation “Align” indicates whether the image only contains faces.
![image/png](https://cdn-uploads.huggingface.co/production/uploads/663f06e01cd68975883a353e/1dbj5KMGyc80Jo0Nyeekd.png)

![image/png](https://cdn-uploads.huggingface.co/production/uploads/663f06e01cd68975883a353e/LeoFyl5yNHhy0xbKQ9BS0.png)
**Image quality score distribution.** (a) BRISQUE evaluation with lower scores indicating better image quality; (b) CLIPIQA evaluation with higher scores indicating better image quality. 

![image/png](https://cdn-uploads.huggingface.co/production/uploads/663f06e01cd68975883a353e/KhNW312RKn8lDsuqFSl92.png)  
**Text distribution.** (a) Distribution of the five categories of annotations in the FaceCaption-15M. (b) The percentage of sentences in the dataset with different word counts. (c) The number of unique 4-grams under the percentage data. (d) Illustrations of image-text pairs LAION-Face and FaceCapition-15M. FaceCaption* indicates the caption generated by grammatical template without using LLM.  

**Note:** The comparison with the CelebV-Text dataset is slightly unfair, as CelebV-Text is a video text description dataset, where we compare the first frame of each video as a picture of the video.   
[CelebV-Text](https://celebv-text.github.io/) is a great dataset, if you need a face video-text dataset, go to the corresponding Github repo. 

# 3. Limitations and Discussions
During our research process, we constructed the FacaCaption-15M dataset. However, in the process of cleaning and producing the dataset, it is inevitable to introduce a certain degree of bias or model prejudice. In response to this, we will persistently update this dataset and strive to minimize the influence of prejudice to the greatest extent.  
In addition, in view of the constraints of relevant laws and regulations such as portrait rights and copyright law, although we have successfully obtained 15 million facial images from LAION, we still decide to follow the open-source release mode of the LAION dataset (that is, to publish the original link of the image, the text description after cleaning, and the position coordinates of the face in the original image).  
Also, if you find that your facial image exists in the dataset and you do not wish your data to be captured, shared, or used for training the model, please contact us. We will conduct a rough review of your information and stop distributing your data in the FaceCaption-15M dataset. It is worth stating that LAION is the upstream of this dataset, and we cannot request the upstream dataset to stop distributing your photos.  
The usage scenarios for large-scale face datasets are limited, and it appears that including wild photos of people holds more research value. We have further cleaned the HumanCaption-15M dataset of human photos in natural scenes based on FaceCaption-15M. Its textual descriptions take into account both scene descriptions and facial details. Stay tuned.  
Due to the special nature of the facial dataset itself, **this dataset is only allowed to be used for scientific research purposes.**

# 4. Contacts
mailto: 2018211556@stu.cqupt.edu.cn or dw_dai@163.com

# 5. Dataset Examples
**The green color is a sample of LAION and the red color is a sample of FaceCaption-15M.**
![image/png](https://cdn-uploads.huggingface.co/production/uploads/663f06e01cd68975883a353e/r9HKtA_ZCRtvIwKIZI4oC.png)

# Additional Information
## Licensing Information
The FaceCaption-15M dataset is released by OpenFaceCQUPT and is intended exclusively for research and educational purposes. It has been generated using publicly available models such as Qwen. Users should be aware that this data may contain inaccuracies, unsafe content, or biases, and should carefully evaluate its accuracy and suitability prior to use. OpenFaceCQUPT and its licensors provide this dataset "AS-IS," without any warranties, express or implied. The views and opinions expressed in the dataset do not necessarily reflect those of OpenFaceCQUPT.

The FaceCaption-15M dataset is licensed under the Creative Commons Attribution 4.0 International License (CC-BY 4.0). The availability of this dataset does not constitute an invitation to use any of the information for any illegal or unlawful purposes, or beyond the scope of research or educational purposes.It is crucial to ensure ethical and responsible use of this dataset to prevent privacy violations and other ethical concerns.

# Citation
```tex
@misc{dai202415mmultimodalfacialimagetext,
      title={15M Multimodal Facial Image-Text Dataset}, 
      author={Dawei Dai and YuTang Li and YingGe Liu and Mingming Jia and Zhang YuanHui and Guoyin Wang},
      year={2024},
      eprint={2407.08515},
      archivePrefix={arXiv},
      primaryClass={cs.CV},
      url={https://arxiv.org/abs/2407.08515}, 
}
```
