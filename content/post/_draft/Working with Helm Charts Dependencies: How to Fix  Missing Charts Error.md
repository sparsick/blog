---
title: 'Working with Helm Charts Dependencies: How to Fix  Missing Charts Error'
date: 
draft: true
tags: ['Cloud', 'helm', 'helm-charts', 'k8s', 'Kubernetes']
---

When you start writing Helm Charts

helm repo add bitnami https://charts.bitnami.com/bitnami  

```
dependencies:
  - name: mongodb
    version: 9.2.2
    repository: https://charts.bitnami.com/bitnami
or
dependencies:
- name: mongodb
  version: 9.2.2
  repository: "@bitnami"

helm install



```

Error: found in Chart.yaml, but missing in charts/ directory

helm dependency update