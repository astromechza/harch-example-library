# Propagate Class

This example demonstrates how resource classes can be propagate via resource references. It involves having a single resource definition that can be parameterized by referencing another resource.

## How the example works

This example is made up of 3 resource definitions. One `s3` and two `config` resource definitions. To keep the examples as simple as possible, the [`humanitec/echo`](https://developer.humanitec.com/integration-and-extensions/drivers/generic-drivers/echo/) driver is used.

The `s3` resource definition [s3-def.yaml](./defs/s3-def.yaml) is configured to match to 2 classes `one` and `two`. It outputs its `bucket` output based on the [Resource Reference](https://developer.humanitec.com/platform-orchestrator/resources/resource-graph/#resource-references) `${resources.config#example.outputs.name}`. This resource reference will cause a new resource to be provisioned and be replaced with the `name` output of that newly provisioned resource. As the resource definition only defines the _type_ of the resource (`config`) and the _ID_ of the resource (`example`), the _class_ that the new resource will be provisioned with will be the same as the class of the `s3` resource.

The one `config` resource definition ([config-one-def.yaml](./defs/config-one-def.yaml)) is configured to match the class `one` and the other [config-two-def.yaml](./defs/config-two-def.yaml) matches the class `two`.

The [score.yaml](./score.yaml) file depends on a resource of type `s3` bucket with a class of `one`. It outputs the bucket name in the environment variable `BUCKET_NAME`.

This means that an S3 bucket will be provisioned via the [s3-def.yaml](./defs/s3-def.yaml) resource definition and the bucket name will be pulled from the resource definition [config-one-def.yaml](./defs/config-one-def.yaml) is configured to match the class `one` and so will be `name-01`.

If the class on the `s3` resource is changed to `two` then the bucket name will be pulled from the resource definition [config-two-def.yaml](./defs/config-two-def.yaml) is configured to match the class `two` and so will be `name-02`.

## Run the demo

### Prerequisites

See the [prerequisites section](/README.md#prerequisites) in the README at the root of this repository.

In addition, the environment variable `HUMANITEC_APP` should be set to `example-propagate-class`.

### Cost

This example will result in a single pod being deployed to a Kubernetes Cluster.

### Deploy the example

1. Create a new app:

   ```bash
   humctl create app "${HUMANITEC_APP}"
   ```

2. Register the resource definitions:

   ```bash
   humctl apply -f ./defs
   ```

3. Deploy the score workload:

   ```bash
   score-humanitec deploy --org "${HUMANITEC_ORG}" --app "${HUMANITEC_APP}" --env "${HUMANITEC_ENV}"
   ```

### Play with the demo

1. In the Humanitec UI, visit the running deployment and look in the container logs for the line starting with `BUCKET_NAME`. It should have a value of `name-01`

2. Change the class of the `s3` resource in the `[score.yaml](./score.yaml)` file from `one` to `two`

3. Redeploy the score file

   ```bash
   score-humanitec deploy --org "${HUMANITEC_ORG}" --app "${HUMANITEC_APP}" --env "${HUMANITEC_ENV}"
   ```

4. In the Humanitec UI, visit the running deployment and look in the container logs for the line starting with `BUCKET_NAME`. It should now have a value of `name-02`

### Clean up the example

1. Delete the application

   ```bash
   humctl delete app "${HUMANITEC_APP}"
   ```

2. Delete the resource definitions

   ```bash
   humctl delete -f ./defs
   ```