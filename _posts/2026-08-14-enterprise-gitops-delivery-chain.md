---
title: "From CI/CD to GitOps: Building an Enterprise Delivery Chain"
description: "A practical delivery architecture from source control to Kubernetes, with selective builds, immutable artifacts, promotion, rollback and release traceability."
categories: [DevOps, Platform Engineering]
tags: [gitops, ci-cd, jenkins, woodpecker, kaniko, harbor, argocd, kubernetes]
---

A deployment pipeline is easy to demonstrate and difficult to operate. The first version usually builds an image and applies a manifest. The production version must answer harder questions: Which source created this image? Who approved the promotion? Can we reproduce the release? What changes when only one service in a monorepo is modified? How quickly can we return to the last known-good state?

GitOps is valuable because it separates artifact production from environment reconciliation and creates a reviewable record of desired state. It does not remove the need for CI; it gives CI a clearer boundary.

## Define the chain and its contracts

One practical enterprise chain is:

Gitea → Jenkins or Woodpecker → Kaniko → Harbor → Git repository → ArgoCD → Kubernetes

Each component should have one clear responsibility.

- Source control stores application code and review history.
- CI validates the change and produces an immutable artifact.
- Kaniko builds container images without depending on a privileged Docker daemon.
- Harbor stores, scans and signs versioned artifacts.
- The deployment repository records the desired version for each environment.
- ArgoCD reconciles Kubernetes with that desired state.
- The cluster reports runtime health through observability systems.

The most important boundary is between producing an artifact and deploying it. A build should not silently mutate production. It should create a traceable candidate that can be promoted through an explicit desired-state change.

## Build once, promote the same artifact

Rebuilding the same commit independently for test and production creates unnecessary uncertainty. Dependencies can change, base images can move and build infrastructure can behave differently.

Build once, assign an immutable digest and promote that exact artifact. Human-readable tags remain useful, but deployment should ultimately resolve to a known image digest.

Release metadata should connect:

- Git commit;
- pipeline execution;
- image repository and digest;
- security scan;
- deployment configuration change;
- Jira release or change record;
- target environment and deployment time.

This chain makes a production incident answerable without reconstructing history from several dashboards.

## Selective builds in a monorepo

A monorepo containing many services can turn every small change into a long and expensive build. Rebuilding all images is simple but wastes time and increases the number of artifacts that appear to have changed.

Selective build starts by identifying the services affected by a commit. Direct changes are straightforward; shared libraries require a dependency map. The pipeline then builds only affected images.

For a release that expects a complete version set, unchanged service images can retain their existing digests while receiving the release metadata required by the deployment model. The important rule is to avoid presenting a retagged artifact as if it were rebuilt from different source.

A robust implementation handles:

- changes to shared packages;
- base-image or build-system changes that affect every service;
- database migrations;
- configuration-only releases;
- manual full-rebuild triggers;
- an auditable list of why each service was or was not rebuilt.

Optimization should never make the release harder to explain.

## Keep secrets out of the pipeline

CI requires credentials to read source, push images and update deployment state. These permissions should be narrow, short-lived where possible and separated by environment.

The build system should not need cluster-admin access. ArgoCD can pull desired state and reconcile through its own controlled identity. Production secrets should not pass through build logs, image layers or repository files.

Use secret scanning before merge, container scanning before promotion and admission policies at deployment. No single control is sufficient: source, artifact and runtime each require an independent boundary.

## Promotion is a policy decision

Development environments may reconcile automatically after a successful build. Production often requires additional evidence: tests, vulnerability thresholds, change windows and an approval from an accountable person.

GitOps represents that decision as a configuration change. A production promotion can be a pull request that updates only the image digest and release metadata. Reviewers see precisely what will change.

Avoid environment drift by keeping configuration inheritance explicit. Overlays and values files should expose differences such as replica counts, resources and external endpoints without duplicating complete manifests.

## Rollback means more than selecting an older image

ArgoCD can return application manifests to a previous Git revision, but data changes may not be backward compatible. A release with a destructive database migration cannot be safely reversed by changing an image tag.

Use expand-and-contract migrations:

1. add backward-compatible schema elements;
2. deploy code that supports old and new structures;
3. migrate data;
4. remove old structures only after the rollback window closes.

Feature flags can decouple deployment from activation. They are especially useful for high-risk integrations, but flags also need ownership and expiry; otherwise they become permanent branches in production behavior.

## Add operational deployment guards

Automation should understand basic production conditions. A deployment can be technically valid and operationally irresponsible if a critical business process is active, an incident is open or the platform has already lost redundancy.

Useful guards include:

- block promotion when required checks are incomplete;
- postpone rollout during critical business activity;
- require healthy storage and database replicas;
- stop when error rate or latency breaches the rollout threshold;
- use progressive delivery for high-impact services;
- notify the responsible team with commit, version and rollback information.

These controls convert operational knowledge into repeatable policy.

## Measure the delivery system

Pipeline success rate alone is not enough. Track lead time, deployment frequency, change-failure rate and recovery time, but interpret them together. A team can increase frequency by creating many low-value deployments, or reduce failures by releasing rarely.

Also measure queue time, build duration by service, cache effectiveness, image size, ArgoCD reconciliation time and the percentage of releases with complete traceability.

Every failed deployment should produce a specific improvement: a better test, a clearer alert, a safer migration pattern or a missing runbook step.

## The outcome

A mature GitOps chain makes releases routine without making them uncontrolled. CI proves and packages a change. The registry preserves the artifact. Git records the desired state. ArgoCD performs reconciliation. Observability verifies the result.

The architecture succeeds when a team can state exactly what is running, why it is running, how it arrived there and how to recover if the release fails.
