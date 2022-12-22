# Installable pluggable components

- Author(s): Marcos Candeia (@mcandeia)
- State: Ready for Implementation
- Updated: 21/12/2022

## Overview

Managing pluggable components became natural with the advent of the [pluggable components injector](0005-R-pluggable-components-injector.md). The pluggable components injector opened new perspectives on how to manage pluggable components by delegating the infrastructure details to the component spec (using annotations that you can specify additional containers), allowing dapr users to focus their efforts only on what their business needs. This makes the separation between three personas clearer: The `dapr operators`, the `dapr users` and the `dapr component developer` (often named as `platform developer`, `app developer` and us `infra/framework developer`). The `dapr component developers` are responsible for building and maintaining dapr components, the `dapr operators` provides the infrastructure to run dapr, whereas the `dapr users` only worry about using them. Dapr operators understand how dapr works in depth, including at component level, and together with the dapr users they architect and build a dapr infrastructure to empower the dapr users in a way that business needs are translated to a specific dapr setup, including which components should be used. Dapr users understand dapr APIs in depth but not at component level, for them, components are seen as the dapr runtime APIs that has a well-defined and documented behavior, whereas dapr operators understand the components internals like dependent services. Dapr component developers understand how the underlying service works in depth while implementing the dapr guarantees backed by the underlying used service.

In addition, the `dapr operators` are also responsible for reusing pluggable components built by the community. Considering that the `dapr users` are the ones that declares components, this is where intersection between a `dapr user`, `dapr operator` and `dapr component develpers` begins, because now, the `dapr users` are responsible for annotating their components spec with the pluggable components annotations, that are provided by the `dapr operators` and built by `dapr component developer`, this adds entropy to the development process as the process of maintaining a pluggable component now is shared among different personas with different development lifecycles. Typically to solve this kind of scenario a communication/bureaucratic process is added to make sure that everything is configured as `dapr operators` expected with the requirements added by `dapr component developer`.

This proposal suggests one more step towards the clear separation between these personas, getting closer to the experience of using a built-in component for the perspective of `dapr users`, and make the `dapr operators` life easier while making the pluggable components community stronger and give them autonomy: **The installable pluggable components**.

## Background

Let's start with a concrete scenario and tell how it would be solved using the desired UX. Imagine a scenario where we have a single app `my-app` using two instances of the same component type `state.my-component` but using different metadata values.

The app deployment/pod:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
name: app
labels:
  app: app
spec:
replicas: 1
selector:
  matchLabels:
    app: app
template:
  metadata:
    labels:
      app: app
    annotations:
      dapr.io/inject-pluggable-components: "true"
      dapr.io/app-id: "my-app"
      dapr.io/enabled: "true"
  spec:
    containers:
      - name: my-app
        image: my-app-image:latest
```

The components:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: my-component-1
  annotations:
    dapr.io/component-container-image: "component:v1.0.0"
    dapr.io/component-container-volume-mounts: "my-component-required-volume;/my-data"
    dapr.io/component-container-env: "MY_ENV_VAR_NAME;MY_ENV_VAR_VALUE"
spec:
  type: state.my-component
  version: v1
  metadata:
    - name: "my-prop"
      value: "my-prop-value"
```

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: my-component-2
  annotations:
    dapr.io/component-container-image: "component:v1.0.0"
    dapr.io/component-container-volume-mounts: "my-component-required-volume;/my-data"
    dapr.io/component-container-env: "MY_ENV_VAR_NAME;MY_ENV_VAR_VALUE"
spec:
  type: state.my-component
  version: v1
  metadata:
    - name: "my-prop"
      value: "my-prop-value-2"
```

#### DRY

When using a component that is pluggable, a container annotation can be optionally added in your components specs. This annotation will ensure that the required container will be present when provisioning the application pod along with daprd container. However, when using multiple instances of the same component, the same setup is repeated, including the container image, volume and volume mounts. Considering the samples above,

Noticed that the container annotations are repeated - which makes sense considering that the same container will be used for all components of the same `type`.

#### Increase the user adoption

#### Creating a strong community

## Related Items

### Related proposals

### Related issues

N/A

## Expectations and alternatives

### What is in scope for this proposal?

### What is deliberately _not_ in scope?

## Implementation Details

### Design

#### Build and publish

#### Components mutating webhook

#### Dapr components CLI commands

##### Install

##### Uninstall

##### List

##### Upgrade

##### Init

##### Conformance-Check

### Feature lifecycle outline

#### Expectations

#### Compatability guarantees

#### Deprecation / co-existence with existing functionality

N/A

### Acceptance Criteria

N/A

## Completion Checklist
