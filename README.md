# Repository Overview

This repository is created by LSA Phyo Thura (lucky). Feel free to share and contribute.

## Table of Contents

1. [Using VPN and How to Access VM](#using-vpn-and-how-to-access-vm)
2. [Installing Kubernetes Cluster](#installing-kubernetes-cluster)
3. [MetalLB and Storage Class](#metallb-and-storage-class)
4. [Installing Ingress-NGINX](#installing-ingress-nginx)
5. [Prometheus Stack](#prometheus-stack)
6. [Traefik](#traefik)
7. [Kafka](#kafka)
8. [Minio](#minio)
9. [Loki](#loki)
10. [Spark](#spark)
11. [Keycloak](#keycloak-and-grafana-integration-guide)
12. [PySpark & Jupyter Notebooks Deployed On Kubernetes](#pyspark--jupyter-notebooks-deployed-on-kubernetes)

## Using VPN and How to Access VM

Instructions on setting up and using a VPN to securely access a virtual machine. For detailed steps, refer to the [Using VPN and Access VM Guide](./01.UsingVPNandAccessVM/readme.md).


## Installing Kubernetes Cluster

A step-by-step guide for installing a Kubernetes cluster, including prerequisites, configuration, and deployment. For detailed steps, refer to the [Installing Kubernetes CLuster](./02.InstallKubernetesCluster/readme.md).

## MetalLB and Storage Class

Setup and configuration of MetalLB for load balancing and creating a storage class for persistent storage in Kubernetes. For detailed steps, refer to the [MetalLB and Storage Class](./03.metalLBandStorageClass/readme.md).

## Installing Ingress-NGINX

How to install and configure NGINX as an ingress controller in a Kubernetes cluster. For detailed steps, refer to the [Installing Ingress-Nginx](./04.Ingress-Nginx/readme.md).

## Prometheus Stack

Deployment and setup of the Prometheus stack for monitoring and alerting in Kubernetes. For detailed steps, refer to the [Installing Prometheus Stack](./05.prometheusStack/readme.md).

## Traefik

Instructions for installing and configuring Traefik as an ingress controller and reverse proxy. For detailed steps, refer to the [Installing Traefik](./06.Traefik/readme.md).

## Kafka

Setting up Apache Kafka for distributed stream processing within a Kubernetes environment. For detailed steps, refer to the [Installing Kafka](./07.Kafka/readme.md).

## Minio

Installation and configuration of Minio for high-performance, S3-compatible object storage in Kubernetes. For detailed steps, refer to the [Installing Minio](./08.minio/readme.md).

## Loki

Guide to installing Loki for log aggregation and monitoring in a Kubernetes cluster. For detailed steps, refer to the [Installing Loki](./09.Loki/readme.md).

## Spark

Deploying an Apache Spark cluster using Helm and running PySpark jobs. Detailed instructions can be found in the `spark` directory. For detailed steps, refer to the [Installing Spark](./10.Spark/readme.md).

### Example Spark Job Deployment

Refer to the `spark/README.md` for a detailed example of deploying a Spark cluster and running a PySpark job. 


### Keycloak and Grafana Integration Guide

This guide provides detailed instructions for integrating Keycloak, an open-source identity and access management solution, with Grafana, a popular open-source platform for monitoring and observability. By following this guide, you will enable Single Sign-On (SSO) and centralized user management for Grafana using Keycloak. For detailed steps, refer to the [Keycloak and Grafana integration](./11.keycloak/README.md).

### PySpark & Jupyter Notebooks Deployed On Kubernetes

 This guide provides detailed instructions for deploying PySpark, a Python API for Apache Spark, and Jupyter Notebooks on a Kubernetes cluster. By following this guide, you will set up a scalable and flexible environment for data processing and analysis using PySpark and Jupyter. For detailed steps, refer to the [PySpark & Jupyter Notebooks Deployed On Kubernetes](./12.%20Jupyter&Pyspark/README.md).
