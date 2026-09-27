# DSO202 Kubernetes Assignments

This repository contains two Kubernetes assignments based on the same Task Tracker application. Both assignments were deployed on a local `kind` cluster.

## Assignment 1: Three Tier Deployment (Unit I)

This assignment deploys the provided frontend, backend, and database images into a dedicated namespace.

The application is connected using ConfigMaps, Secrets, Services, and a PersistentVolumeClaim.

It covers namespace setup, configuration and secrets, deployment of all three tiers, resource management, and full application verification.

The verification includes the CRUD cycle, DNS resolution, self healing, and declarative and imperative Kubernetes operations.

Full report: [Assignment1_Report.md](Assignment1_Report.md)

## Assignment 2: Unit II Concepts Applied

This assignment extends the same Task Tracker deployment with four advanced Kubernetes concepts from Unit II.

The concepts covered are RBAC for limited access, Ingress for routing traffic through a single entry point, StatefulSets with a three replica PostgreSQL demonstration for stable identity and storage, and Operators using a CRD and Custom Resource.

The Operator demonstration also shows that a Custom Resource does not manage an application by itself without a controller.

Full report: [Assignment2_Report.md](Assignment2_Report.md)

## Repository Structure

```text
├── README.md
├── Assignment1_Report.md
├── Assignment2_Report.md
├── namespace.yaml
├── configmap.yaml
├── secret.yaml
├── quota.yaml
├── database/
├── backend/
├── frontend/
├── rbac/
├── ingress/
├── unit2-demos/
│   ├── statefulset/
│   └── operator/
├── evidence/
└── evidence-u2/

The evidence folder contains screenshots from Assignment 1, while evidence-u2 contains screenshots from Assignment 2.