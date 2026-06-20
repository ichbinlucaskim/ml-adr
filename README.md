# ml-adr


| Paradigm        | Learns from          | Final output                                    |
| --------------- | -------------------- | ----------------------------------------------- |
| Supervised      | human labels         | a prediction                                    |
| Unsupervised    | data structure alone | groups / compressed features / scores / samples |
| Self-supervised | fabricated labels    | a transferable representation                   |
| Reinforcement   | reward from acting   | a policy (strategy)                             |


- one piece of intuition to keep in mind: industry choices usually come down to **"do you have labels, can you make them, and how expensive are they?"**: Most real-world projects start with supervised, then mix in self-supervised pre-training or unsupervised techniques when labels run short.

```
Labels abundant/cheap        → Supervised (try this first)
No labels / expensive        → Unsupervised (structure discovery, anomaly detection)
Labels expensive but data is a mountain → Self-supervised foundation, then fine-tune with few labels
Answer comes only as "the result of an action" → Reinforcement
```

```
┌──────────────────────────────────────────────────────────────┐
│  ML Task Map (by Learning Paradigm)                          │
└──────────────────────────────────────────────────────────────┘
[A] Supervised (labels y available)  ← the pipelines built so far
    │
    ├─ Classification (discrete labels)
    │     ├─ Binary        (2 classes)       
    │     ├─ Multiclass    (1 of K)          
    │     └─ Multilabel    (several at once)
    │
    └─ Regression (continuous values)        
          e.g., price, temperature, demand forecasting
[B] Unsupervised (no labels)  ← clustering belongs here
    │
    ├─ Clustering
    │     K-means, DBSCAN, GMM, hierarchical
    │     (DL version: Deep Embedded Clustering, etc.)
    │
    ├─ Dimensionality Reduction
    │     PCA, t-SNE, UMAP
    │     (DL version: Autoencoder)
    │
    ├─ Density Estimation / Anomaly Detection
    │     outlier detection
    │
    └─ Generative
          learn the distribution to generate new data
          (DL version: GAN, VAE, Diffusion, LLM)
[C] Self-supervised (data creates its own labels)
    mask part of the data and predict it → representation learning
    e.g., BERT (fill-in-the-blank), contrastive learning
    → the pre-training approach behind today's large models
[D] Reinforcement Learning (learns from reward)
    interact with an environment → learn a policy that maximizes reward
    e.g., games, robot control, RLHF
```

---

```
┌──────────────────────────────────────────────────────────────┐
│  ML Task Map — Real-World Applications                       │
└──────────────────────────────────────────────────────────────┘
```

**[A] Supervised — "Learn from past answers to predict the future"**

The most lucrative and most widely used branch. Labeled historical data is the core asset.

*Binary classification* — Yes/no decisions. The most common in industry. Spam filters (spam or not), loan default prediction (will they repay or not), credit card fraud detection (fraud or legitimate), churn prediction (will this customer cancel next month), ad click prediction (will they click or not), disease diagnosis (positive/negative). In business, almost any "is this person/case X?" lands here.

*Multiclass* — One out of several bins. Auto-routing customer inquiries (refund / shipping / tech support), product category classification from images, sentiment analysis (positive/negative/neutral), handwriting and license-plate recognition.

*Multilabel* — Several apply at once. News article tagging (one article is 'politics + economy + international' simultaneously), multiple object labels in an image (one photo has 'dog + grass + frisbee'), movie genres (action + comedy at once). Easy to confuse with multiclass — "pick exactly one" is multiclass, "several switched on" is multilabel.

*Regression* — Predicting the number itself. Real-estate price estimation, power/demand forecasting, ad bid optimization, delivery ETA prediction, inventory demand forecasting, insurance premium calculation. If the answer is "how much / how many / how many minutes," it's here.

**[B] Unsupervised — "Discover structure in data without answers"**

Used when labels are expensive or impossible to obtain, or when you don't yet know what's in the data.

*Clustering* — Grouping similar things together. The flagship case is customer segmentation — automatically sorting customers into 'VIP / discount-sensitive / dormant' groups by purchase pattern, then applying different marketing to each. Also: grouping similar users for recommender systems, auto-grouping documents/news by topic, store location analysis. The key point is that **there's no predefined answer for who belongs to which group.**

*Dimensionality Reduction* — Compressing hundreds of variables into 2–3. Mainly used for visualization (plotting high-dimensional data in 2D to see patterns) and preprocessing (removing noise/redundancy before feeding another model). Gene expression analysis and image embedding compression are typical.

*Anomaly Detection* — Finding "what deviates from normal." This is the core of fields with almost no labels. Predictive maintenance for factory equipment (sensor values differ from usual → failure coming), server intrusion/hacking detection, credit card anomalies (solved with supervised too, but since fraud cases are so rare, unsupervised methods catch "unusual patterns"), quality inspection.

*Generative* — Learning a distribution to create something new. The most talked-about branch right now. Image generation (product mockups, ad creatives), text generation (LLM chatbots, copywriting), voice synthesis (TTS), novel drug-candidate molecule generation, data augmentation (creating synthetic training data).

**[C] Self-supervised — "Data quizzes itself to build 'intuition'"**

In industry, rather than being the final product itself, this is mostly used as the **pre-training stage of large models.** You first build representational power on massive unlabeled data (web text, images), then fine-tune with a small amount of labels. The classic case is LLMs like ChatGPT acquiring a sense of language by predicting blanks/next words in internet text. On the image side, contrastive learning (two augmentations of the same photo should be close, different photos far apart) is here too. Its industry meaning is "a foundation that dramatically cuts labeling cost."

**[D] Reinforcement Learning — "Learn strategy through trial and error"**

Maximizes reward by interacting with an environment. Industrial application is narrower than the other three, but there are powerful niches. Sequential decision-making in recommendation/advertising (what to show next), robot control and autonomous driving, logistics/warehouse route optimization, process/energy control (data-center cooling optimization is famous), and **RLHF for LLMs** (using human preference as the reward signal to refine answers — the process that makes Claude or ChatGPT "follow human intent well").