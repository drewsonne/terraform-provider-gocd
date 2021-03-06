# Version Migration

This document describes any changes that you must make to your HCL to migrate to any version that introduces breaking changes.

## 1.0.0

This version removes the `gocd_pipeline_stage` resource and replaces it with the `gocd_stage_definition` data source.

Also `gocd_pipeline_stage.manual_approval` has been replaced by `gocd_stage_definition.approval.type` (mirroring the GoCD API).

**How to update your terraform:**

Find all occurrences of `resource "gocd_pipeline_stage"` in code and replace these with `data "gocd_stage_definition"`.

Remove the `pipeline` property from the stage definitions, instead creating the link between stages and pipelines using a `stages = [...]` property in every `gocd_pipeline` and `gocd_pipeine_template` resource.

Replace any lookup of `${gocd_pipeline_stage.name.property}` with `${data.gocd_stage_definition.name.property}`

Replace any uses of `gocd_pipeline_stage.manual_approval` with `approval { type = "manual" }`

**Example**:

The following terraform to produce 1 pipeline with 1 stage:

```
resource "gocd_pipeline" "example" {
   name = "example"
   group = "example"
   materials = [...]
}

resource "gocd_pipeline_stage" "example" {
   name = "example"
   pipeline = "${gocd_pipeline.pipeline.name}"
   jobs = [...]
   manual_approval = true
}
```

Now becomes:

```
resource "gocd_pipeline" "example" {
   name = "example"
   group = "example"
   materials = [...]
   stages = ["${data.gocd_stage_definition.example.json}"]
}

data "gocd_stage_definition" "example" {
   name = "example"
   jobs = [...]
   approval {
     type = "manual"
   }
}
```

If you have existing state, you must first remove all `gocd_pipeline_stage` resources (this will not modify any pipelines):
```
terraform refresh
terraform state list | grep gocd_pipeline_stage | xargs terraform state rm
```
