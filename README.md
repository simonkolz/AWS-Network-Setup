# AWS Projects README

## Overview

This repository contains two AWS infrastructure projects demonstrating practical cloud engineering skills with Amazon Web Services. Both projects were built from the ground up using industry best practices for security, scalability, and high availability.

---

## 📁 Project 1: AWS VPC Network Setup

In this project, I designed and deployed a fully secure Virtual Private Cloud (VPC) architecture across multiple Availability Zones. This foundational networking setup includes:

* **Multi-AZ VPC** configuration for high availability
* **Public and Private Subnets** split across two Availability Zones
* **Internet Gateway** for controlled ingress/egress
* **NAT Gateway** to allow private instances secure outbound access
* **Bastion Host (Jump Box)** for secure administrative access to private EC2 instances
* **Route Tables & Security Groups** designed with the principle of least privilege

This architecture provides a production-ready network environment suitable for secure workloads and scalable applications.

---

## 📁 Project 2: Application Load Balancer Deployment

This project focuses on deploying an **Application Load Balancer (ALB)** to distribute traffic across EC2 instances in multiple Availability Zones.

Key components include:

* **ALB spanning multiple subnets** for fault tolerance
* **Target Groups** routing traffic to EC2 instances
* **Health Checks** for automatic detection of unhealthy instances
* **Security Groups** ensuring controlled, secure traffic flow
* **Auto Scaling compatibility** for future expansion if needed

The result is a highly available, resilient, and secure load-balanced web architecture.

---

## 🎯 Purpose

These projects demonstrate hands-on experience with:

* AWS networking fundamentals
* Secure cloud architecture design
* High availability and multi-AZ deployments
* Load balancing and traffic management

---

## 🚀 Technologies Used

* Amazon VPC
* EC2
* Application Load Balancer (ALB)
* NAT Gateway
* Bastion Host
* IAM
* Route Tables, Subnets, Security Groups
