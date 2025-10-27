# October 27th Meeting - Linear probing on MLLM (Llava 1.5)

# Recap last meeting:

Did Linear probing on the ViT layers of CLIP:

(Hyperparameters were not optimized for best possible result, big learning rate is used for faster training, not too many epochs)

CLIP L14 is giving me Val mIoU 0.3330

CLIP B16 yields less: 0.31 mIoU

These results include an optimization, that other benchmarks did, including the CLS token for segmentation by concatenating it to the patches.

The MLLM tokens look like this [602:4096] in this case for Llava 1.5:

['<s>', '▁US', 'ER', ':', '▁', '<image>', '<image>', '<image>', '<image>', '<image>', '<image>', '<image>', '<image>', '<image>', '<image>', '<image>’,[….576 image tokens in total] ‘<image>',, '▁', '<0x0A>', 'Ident', 'ify', '▁regions', '▁that', '▁correspond', '▁to', '▁objects', '.', '▁Descri', 'be', '▁each', '▁patch', '▁briefly', '.', '<0x0A>', 'ASS', 'IST', 'ANT', ':']

We just take the image tokens only and linear probe each layer.

Linear Probing for Segmentation on MLLM Layers is unconventional.

# This meeting: Linear probing on MLLM (Llava 1.5)

## Todo for this meeting:

1. General prompt to just find the best layer for segmentation, then use that layer for next step
2. Use that layer to try specific prompt and see how that performance decreases or increases

Use the following prompts:

For the 1. Task “**Describe the image in detail”**

For the 2. Task “**Describe the [category] in the image in detail”**

(Hmm, actually according to the paper “Words That Make Language Models Perceive” we should use a prompt like this for the second task: ”**Imagine what it would look like to see A [category]”**)

## Probing CLIP again to set more comparable baseline

We previously probed CLIP L14 with concatenation of CLS token which was 0.33mIou, but Llava uses CLIP L14@336 and they drop the CLS token, so:

**CLIP L14 without CLS token:**

Epoch 001 | loss 2.3412 | train mIoU 0.2091 acc 0.5839 | test mIoU 0.2042 acc 0.5805
Epoch 002 | loss 1.4925 | train mIoU 0.2766 acc 0.6232 | test mIoU 0.2593 acc 0.6161
Epoch 003 | loss 1.3208 | train mIoU 0.3055 acc 0.6390 | test mIoU 0.2813 acc 0.6302
Epoch 004 | loss 1.2458 | train mIoU 0.3222 acc 0.6484 | test mIoU 0.2900 acc 0.6382
Epoch 005 | loss 1.2059 | train mIoU 0.3358 acc 0.6526 | test mIoU 0.2971 acc 0.6407
Epoch 006 | loss 1.1672 | train mIoU 0.3435 acc 0.6572 | test mIoU 0.3015 acc 0.6442
Epoch 007 | loss 1.1573 | train mIoU 0.3509 acc 0.6602 | test mIoU 0.3044 acc 0.6456
Epoch 008 | loss 1.1382 | train mIoU 0.3565 acc 0.6620 | test mIoU 0.3054 acc 0.6473
Epoch 009 | loss 1.1270 | train mIoU 0.3584 acc 0.6634 | test mIoU 0.3041 acc 0.6479
Epoch 010 | loss 1.1214 | train mIoU 0.3637 acc 0.6655 | test mIoU 0.3077 acc 0.6492
Epoch 011 | loss 1.1136 | train mIoU 0.3677 acc 0.6667 | test mIoU 0.3091 acc 0.6502
Epoch 012 | loss 1.1062 | train mIoU 0.3726 acc 0.6677 | test mIoU 0.3121 acc 0.6500
Epoch 013 | loss 1.1044 | train mIoU 0.3759 acc 0.6684 | test mIoU 0.3116 acc 0.6509
Epoch 014 | loss 1.0954 | train mIoU 0.3786 acc 0.6697 | test mIoU 0.3144 acc 0.6515
Epoch 015 | loss 1.0950 | train mIoU 0.3776 acc 0.6701 | test mIoU 0.3103 acc 0.6512

**mIoU of clip l14 without CLS token was around 0.31**

## Now on clip L14@336

**mIoU of clip l14@336 around 0.29**

max: Test mIoU: 0.3012 | Pixel Acc: 0.6071

![image.png](October%2027th%20Meeting%20-%20Linear%20probing%20on%20MLLM%20(Lla/image.png)

Could have reached 30% with longer training:

![image.png](October%2027th%20Meeting%20-%20Linear%20probing%20on%20MLLM%20(Lla/image%201.png)

![output.png](October%2027th%20Meeting%20-%20Linear%20probing%20on%20MLLM%20(Lla/output.png)

Epoch 001 | loss 2.7653 | train mIoU 0.1917 acc 0.5302 | test mIoU 0.1887 acc 0.5268
Epoch 002 | loss 1.9456 | train mIoU 0.2539 acc 0.5673 | test mIoU 0.2455 acc 0.5633
Epoch 003 | loss 1.7592 | train mIoU 0.2812 acc 0.5836 | test mIoU 0.2644 acc 0.5783
Epoch 004 | loss 1.6664 | train mIoU 0.2992 acc 0.5939 | test mIoU 0.2775 acc 0.5858
Epoch 005 | loss 1.6198 | train mIoU 0.3110 acc 0.5994 | test mIoU 0.2835 acc 0.5912
Epoch 006 | loss 1.5736 | train mIoU 0.3136 acc 0.6009 | test mIoU 0.2834 acc 0.5910
Epoch 007 | loss 1.5504 | train mIoU 0.3266 acc 0.6041 | test mIoU 0.2920 acc 0.5937
Epoch 008 | loss 1.5252 | train mIoU 0.3290 acc 0.6074 | test mIoU 0.2926 acc 0.5967
Epoch 009 | loss 1.5010 | train mIoU 0.3320 acc 0.6071 | test mIoU 0.2935 acc 0.5960
Epoch 010 | loss 1.5004 | train mIoU 0.3368 acc 0.6131 | test mIoU 0.2953 acc 0.6014
Epoch 011 | loss 1.4815 | train mIoU 0.3408 acc 0.6132 | test mIoU 0.2975 acc 0.6011
Epoch 012 | loss 1.4702 | train mIoU 0.3421 acc 0.6171 | test mIoU 0.2974 acc 0.6059
Epoch 013 | loss 1.4632 | train mIoU 0.3442 acc 0.6173 | test mIoU 0.2964 acc 0.6035
Epoch 014 | loss 1.4539 | train mIoU 0.3482 acc 0.6178 | test mIoU 0.2999 acc 0.6051
Epoch 015 | loss 1.4472 | train mIoU 0.3503 acc 0.6202 | test mIoU 0.3012 acc 0.6071

**Why is quality worse even if we have larger input?**

Maybe:

- ViT CLIP L14 @336 is just the normal L14 trained:
    
    *For the ViT-L/14 we also pre-train at a higher 336
    pixel resolution for one additional epoch to boost perfor-
    mance similar to FixRes (Touvron et al., 2019). We denote
    this model as ViT-L/14@336px. (Clip paper page 5)*
    
- Other paper saw this too:
    
    “Image Resolution for Zero-Shot Classification We report the evaluation results for both SAM-CLIP and CLIP models using the 224px image resolution. However, we found that SAM-CLIP benefits from the 336px resolution, **whereas
    the performance of CLIP models deteriorates (they exhibit worse accuracy) (**[https://arxiv.org/pdf/2310.15308](https://arxiv.org/pdf/2310.15308) Page 15**)**
    

## Large focus was making the Data and Train pipeline work with unprecedented large amounts of data

Problem: The entire ADE20k dataset, including train and validation is about 20k+2k images, on disk that is less than 1GB of space.

**But the hidden features of a single MLLM layer are already almost 100GB on disk, only loaded to RAM thats about 150GB, so we are dealing with 150x the data:**

Total RAM: 1081.82 GB (Entire Server)
**Used RAM: 152.85 GB**

What I did:

- Implemented a dataset sharding method, that reduces memory usage, by only loading one .pt file at a time, that is 2GB, caching the features. (It basically scans all the pt files, remembers an index and then for each index it knows in which .pt file to look, loads that one and returns the feature with the corresponding index.)
- Implement a Dataloader that reads data shard by shard (this turns out to be a bottleneck later)
- Reimplemented training pipeline to work with the dataset and loader.

After that worked I started training, but the issue was that it was extremely slow.

I started trying to diagnose the issue and later found it:

It takes about 25 seconds to load one single batch… The batch size was 64 (its small compared to 22k images in one epoch including validation)

The issue was, IO bottleneck, training is done with suffling the dataset, so it would look inside the 41 files, each over 2GB, to pull out one single feature, each batch in the worst case creates a 140 GB of IO…

A modern NVMe SSD (PCIe 3.0)
Typical read speed: 3 GB/s (theoretical)
Time = 140 GB ÷ 3 GB/s → 46.7 s
≈ 45–50 seconds on a fast NVMe SSD

I did not find a solution to this, since training with so much data creates a bunch of new issues. I am not familiar with training with large amounts of data, maybe its normal that its this slow.

My solution then was to not shuffle the dataset and load them shard by shard:

Train improvement is much slower, and training a model without shuffling is generally a bad approach…

Epoch 001 | loss 3.3480 | train mIoU 0.0953 acc 0.3819 | test mIoU 0.0950 acc 0.3765
Epoch 002 | loss 2.4068 | train mIoU 0.1875 acc 0.4820 | test mIoU 0.1860 acc 0.4764
Epoch 003 | loss 2.1157 | train mIoU 0.2297 acc 0.5212 | test mIoU 0.2239 acc 0.5147
Epoch 004 | loss 1.9782 | train mIoU 0.2532 acc 0.5413 | test mIoU 0.2427 acc 0.5337
Epoch 005 | loss 1.8941 | train mIoU 0.2692 acc 0.5533 | test mIoU 0.2543 acc 0.5451
Epoch 006 | loss 1.8374 | train mIoU 0.2808 acc 0.5616 | test mIoU 0.2627 acc 0.5530
Epoch 007 | loss 1.7886 | train mIoU 0.2900 acc 0.5683 | test mIoU 0.2691 acc 0.5593
Epoch 008 | loss 1.7544 | train mIoU 0.2972 acc 0.5731 | test mIoU 0.2729 acc 0.5638
Epoch 009 | loss 1.7240 | train mIoU 0.3031 acc 0.5771 | test mIoU 0.2763 acc 0.5675
Epoch 010 | loss 1.6971 | train mIoU 0.3080 acc 0.5805 | test mIoU 0.2791 acc 0.5707
Epoch 011 | loss 1.6756 | train mIoU 0.3122 acc 0.5834 | test mIoU 0.2819 acc 0.5734
Epoch 012 | loss 1.6572 | train mIoU 0.3157 acc 0.5860 | test mIoU 0.2836 acc 0.5757
Epoch 013 | loss 1.6431 | train mIoU 0.3190 acc 0.5883 | test mIoU 0.2850 acc 0.5776
Epoch 014 | loss 1.6284 | train mIoU 0.3216 acc 0.5901 | test mIoU 0.2863 acc 0.5791
Epoch 015 | loss 1.6155 | train mIoU 0.3239 acc 0.5919 | test mIoU 0.2878 acc 0.5806

![output_shuffle_off.png](October%2027th%20Meeting%20-%20Linear%20probing%20on%20MLLM%20(Lla/output_shuffle_off.png)

## The final solution was to request huge amounts of RAM

By requesting hundreds of GB of CPU Ram, i was able to load the entire dataset once into RAM, still shuffle and train without creating a lot of disk IO. It finally worked on MLLM layers.

# A Brief detour about the LRZ Cluster

Some interesting stuff I realized about the LRZ cluster:

## Your Job Priority: How LRZ wrongly calculates your usage and how it is easily tricked…

How fast a user job is processed is based on their fairshare, which is based on their total usage which is based on their billing for each job.

But the way the LRZ cluster is set up, was not quite thought through, it seems like a basic setting was used.

If we look at the following 2 jobs:

Job 1:
AllocTRES=cpu=16,mem=364G,node=1,billing=745584,gres/gpu=16
TresPerNode=gres/gpu:16

Job 2 (my job):
AllocTRES=cpu=8,mem=128G,node=1,billing=262165,gres/gpu=1
TresPerNode=gres/gpu:3g.40gb

Job 1 uses 16 GPUs while job 2 uses 3/7 of one GPU, so not even one. Why is the billing for the first 16 GPU job only 2.8x larger?

Because the LRZ Cluster weights assigned CPU RAM way way more than anything else, like GPUs.

**→ It doesn’t really matter what kind of GPU you use and how many, what matters way more is the Amount of RAM you assigned in total. So try to keep RAM as low as possible. Overwrite the default RAM assignment to something lower if possible.**

## The biggest abuser of the LRZ Cluster: di97fer

Account NormShares NormUsage
di97fer     0.000605       0.087005

Saw this person in front of my queue often and checked out his account: This person is using 143 times his assigned compute consistently, by queuing his jobs for a long time he gets to run them, since jobs that waited for 2 days will get priority. Last night i checked 2 of his jobs, they were using 16 GPUs each, and were interactive bash jobs, that ran until the 48h timeout. Very weird…

## Sanghwan’s jobs are too “nice”

I just checked something you sent me, your bashrc file, where you define jobs the following way:
srun --partition=mcml-hgx-h100-94x4 --qos=mcml --nice=10000 --gres=gpu:1 --nodes=1 --time=1:00:00 --mem=40G --pty bash

**→ Not sure whether you know this and do this on purpose but: By setting a high nice value, your job gets deprioritized, causing you to wait longer. 10000 nice is a high value, without that value, or a 0, you would get assigned a node a little faster, which might be helpful to you since this is an interactive bash request.**

okay back to results

# Extracting feature tokens first layer

I am efficiently using resources, full memory and wattage for feature extraction:

![image.png](October%2027th%20Meeting%20-%20Linear%20probing%20on%20MLLM%20(Lla/image%202.png)

But having Issues optimizing training of the linear probe with full GPU util, didnt manage to get wattage to full 400W, maybe because the linear probe is just a one layer few para model and its more about loading data, which is the bottleneck.

But requesting over 100GB of RAM, while not disturbing anyone since there is plenty available, nor using a lot of electricity since I only use 1 GPU, my “fairshare” is totally trash now and I have been deprioritized lol, anyways:

## Linear Probe first layer result

![image.png](October%2027th%20Meeting%20-%20Linear%20probing%20on%20MLLM%20(Lla/image%203.png)

![image.png](October%2027th%20Meeting%20-%20Linear%20probing%20on%20MLLM%20(Lla/image%204.png)

Epoch 001 | loss 2.3701 | train mIoU 0.2195 acc 0.5189 | test mIoU 0.2127 acc 0.5148
Epoch 002 | loss 2.0016 | train mIoU 0.2486 acc 0.5448 | test mIoU 0.2383 acc 0.5399
Epoch 003 | loss 1.8927 | train mIoU 0.2575 acc 0.5532 | test mIoU 0.2415 acc 0.5462
Epoch 004 | loss 1.8325 | train mIoU 0.2652 acc 0.5540 | test mIoU 0.2480 acc 0.5463
Epoch 005 | loss 1.7847 | train mIoU 0.2736 acc 0.5668 | test mIoU 0.2532 acc 0.5592
Epoch 006 | loss 1.7506 | train mIoU 0.2814 acc 0.5733 | test mIoU 0.2580 acc 0.5653
Epoch 007 | loss 1.7193 | train mIoU 0.2828 acc 0.5736 | test mIoU 0.2581 acc 0.5655
Epoch 008 | loss 1.6984 | train mIoU 0.2856 acc 0.5757 | test mIoU 0.2616 acc 0.5673
Epoch 009 | loss 1.6781 | train mIoU 0.2943 acc 0.5767 | test mIoU 0.2665 acc 0.5676
Epoch 010 | loss 1.6624 | train mIoU 0.2938 acc 0.5790 | test mIoU 0.2662 acc 0.5691
Epoch 011 | loss 1.6541 | train mIoU 0.2986 acc 0.5816 | test mIoU 0.2707 acc 0.5731
Epoch 012 | loss 1.6353 | train mIoU 0.2995 acc 0.5806 | test mIoU 0.2697 acc 0.5707
Epoch 013 | loss 1.6230 | train mIoU 0.3004 acc 0.5834 | test mIoU 0.2724 acc 0.5761
Epoch 014 | loss 1.6126 | train mIoU 0.3026 acc 0.5835 | test mIoU 0.2719 acc 0.5750
Epoch 015 | loss 1.6038 | train mIoU 0.3037 acc 0.5908 | test mIoU 0.2731 acc 0.5813
Epoch 016 | loss 1.6015 | train mIoU 0.3039 acc 0.5825 | test mIoU 0.2702 acc 0.5732
Epoch 017 | loss 1.5899 | train mIoU 0.3051 acc 0.5909 | test mIoU 0.2728 acc 0.5817
Epoch 018 | loss 1.5832 | train mIoU 0.3092 acc 0.5918 | test mIoU 0.2750 acc 0.5815
Epoch 019 | loss 1.5809 | train mIoU 0.3051 acc 0.5912 | test mIoU 0.2731 acc 0.5811
Epoch 020 | loss 1.5774 | train mIoU 0.3057 acc 0.5885 | test mIoU 0.2709 acc 0.5781

![image.png](October%2027th%20Meeting%20-%20Linear%20probing%20on%20MLLM%20(Lla/image%205.png)

Final:
**Test mIoU: 0.2750 | Pixel Acc: 0.5815**

Okay that was our first layer, lets probe the following 10 layers:

1,2,4,8,12,16,20,24,28,31,32

Its actually kind of tedious because each layer takes over an hour. Getting feedback on final code changes takes like one hour until I get a result, and when I thought everything was working, Slurm killed my job before any result was saved because the default 1h limit was reached…

# Okay here the final result