---
title: 'My Preparation for CKAD Exam'
date: 2021-12-02
description: "Certified Kubernetes Application Developer, CKAD, experience, preparation" # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
#codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Kubernetes
  - Reading-Tips
tags:
  - Kubernetes
  - Certification
# comment: false # Disable comment if false.
---

Some days ago, I passed my [CKAD (Certified Kubernetes Application Developer) exam](https://www.credly.com/badges/7848d923-82cf-42be-a226-b20457cf981e).
In this post I'd like to share the resources, that I used for my preparation.

I started with reading the book [Kubernetes in Action, First Edition](https://www.manning.com/books/kubernetes-in-action-second-edition) to understand the concept of Kubernetes (k8s) under the hood.
It gave me a deep dive in k8s.
It was a deeper dive than I really need as a developer in my daily life, but this knowledge helps me when I have to do troubleshooting and in the communication with k8s administrators.

I decided to do the CKAD exam because of a special offer of the [Linux Foundation](https://www.linuxfoundation.org/).
They had a discount campaign for their [Kubernetes for Developers (LFD259) + CKAD Exam Bundle](https://training.linuxfoundation.org/training/kubernetes-for-developers-lfd259-ckad-exam-bundle/).
The training was nice for understanding the content of the exam.
But I don't think the training would be enough for exam preparation.

When you read blog post about experiences with the KCAD exam, then the main point is that you have to be fast on the CLI.
Therefore, I searcht for more exercises.
I chose [CKAD Exercises by Dimitris-Ilias Gkanatsios] and [CKAD Practice Tests on Katacode test environment by Liptan Biswas](https://dev.to/liptanbiswas/ckad-practice-questions-4mpn) for the training of my muscle memory.
Train it with [`kubectl` auto-completion](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-autocomplete)
Furthermore, I learnt some VIM commands, because you need to be fast in an editor and VIM is pre-installed on every environment I used for my preparation.

The VIM commands that help me:
- `:q` - quit
- `:wq` - save and quit
- `i` - insert before the cursor
- `yy` - yank (copy) a line
- `2yy` - yank (copy) 2 lines (replace 2 by a number of your choise)
- `p` - put (paste) the clipboard after cursor

The environment for the exercises in the training and for the CKAD Exercises by Dimitris-Ilias Gkanatsios I used a self-installed k8s cluster on the Google Cloud.
The training has a tutorial how to install a two node cluster in AWS or Goggle Cloud.
I think a local minikube should also be enough for practicing.

After finishing my muscle memory training, I took a session on the [Kubernetes Exam Simulator](https://killer.sh/).
After you register your exam, you will get two free sessions on it.
It is a test environment which comes really close to the original one.
The questions have to be solved in 120 minutes, and they are harder than the exam question, so it's a good final test about your preparation status.
After I finished the simulator, I had a good feeling that I'm prepared for the real exam.

The simulator showed me that I had some understanding problems with two topics.
I skimmed through the slides of [CKAD Crash Course by Benjamin Muschko](https://github.com/bmuschko/ckad-crash-course).
There are exercises., too.
For the case, you need more training.
For me, these would have been my backup training, if I hadn't passed the exam.


## Summmary (TL;DR)

- The book [Kubernetes in Action, First Edition](https://www.manning.com/books/kubernetes-in-action-second-edition) helps me to Kubernetes under the hood.
- [Kubernetes for Developers (LFD259) + CKAD Exam Bundle](https://training.linuxfoundation.org/training/kubernetes-for-developers-lfd259-ckad-exam-bundle/) of Linux Foundation was available in a special offer. Therefore,  I chose it as training.
- [Set of CKAD Exercises by Dimitris-Ilias Gkanatsios](https://github.com/dgkanatsios/CKAD-exercises) that help me to practice my muscle memory.
- [CKAD Practice Tests on Katacode test environment by Liptan Biswas](https://dev.to/liptanbiswas/ckad-practice-questions-4mpn) that is a good free simulation of the real exam environment and trains also my muscle memory.
- Learn some VIM commands.
- Use [`kubectl` auto-completion](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-autocomplete)
- [Kubernetes Exam Simulator](https://killer.sh/) (two sessions are free over the exam registration) gave me a good feeling that I'm prepared.
- [CKAD Crash Course by Benjamin Muschko](https://github.com/bmuschko/ckad-crash-course) has more CKAD exercise and slides about the exam content. I only read the slides.
