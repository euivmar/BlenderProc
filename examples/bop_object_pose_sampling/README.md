## Benchmark for 6D Object Pose Estimation (BOP) <br/> Sampling objects, cameras and lights

<img src=tless_sample.png width="240" height="180"> <img src=hb_sample.png width="240" height="180"> <img src=hb_sample_inst.png width="240" height="180">

This example shows how to load BOP objects and alternatingly sample light poses, camera poses (looking towards the objects) and object poses (including collision checks).

## Usage

First make sure that you have downloaded a [BOP dataset](https://bop.felk.cvut.cz/datasets/) in the original folder structure. Also please clone the [BOP toolkit](https://github.com/thodan/bop_toolkit).

In [examples/bop_object_pose_sampling/config.yaml](config.yaml) set the `blender_install_path` where Blender is or should be installed.

Execute in the BlenderProc main directory:  

```
python run.py examples/bop_object_pose_sampling/config.yaml <path_to_bop_data> <bop_dataset_name> <path_to_bop_toolkit> examples/bop_object_pose_sampling/output
```
* `examples/bop_object_pose_sampling/config.yaml`: path to the pipeline configuration file.
* `<path_to_bop_data>`: path to a folder containing BOP datasets.
* `<bop_dataset_name>`: name of BOP dataset, e.g. lm
* `<path_to_bop_toolkit> `: path to the BOP toolkit containing dataset parameters, etc.
* `examples/bop_object_pose_sampling/output`: path to the output directory.

## Visualization

Visualize the generated data and labels:
```
python scripts/visHdf5Files.py examples/bop_object_pose_sampling/output/0.hdf5
```

## Steps

* Loads object models and camera intrinsics from specified BOP dataset: `loader.BopLoader` module.
* Creates a point light sampled inside a shell: `lighting.LightSampler` module.
* Loops over: `composite.CameraObjectSampler` module.
    * Sample Object Poses inside a cube with collision checks
    * Sample Camera Poses inside a shell looking at objects
* Renders rgb: `renderer.RgbRenderer` module.
* Renders instance segmentation masks: `renderer.SegMapRenderer` module.
* Writes instance segmentation masks: `writer.CocoAnnotationsWriter` module.
* Writes labels and images to compressed hdf5 files in output_dir: `writer.Hdf5Writer` module.
* Writes BOP labels: `writer.BopWriter` module.

## Config file

### BopLoader

If `scene_id` is not specified (default = -1), `loader.BopLoader` simply loads all or the specified `obj_ids` from the BOP dataset given by `bop_dataset_path`. 

```yaml
    {
      "name": "loader.BopLoader",
      "config": {
        "bop_dataset_path": "<args:0>/<args:1>",
        "mm2m": True,
        "split": "val",
        "obj_ids": [1,1,3],
        "model_type": ""
      }
    },
```

Following modules handle the sampling.

 `composite.CameraObjectSampler` alternates between sampling new cameras using a `camera.CameraSampler` and sampling new object poses using a `object.ObjectPoseSampler`. Additionally, here you set the parameters

- `noof_cams_per_scene` after which the object poses are resampled
- `total_noof_cams` to generate

Alternatively, since we generated COCO annotations, you can also visualize the generated coco_annotations.json file:
```
python scripts/vis_coco_annotation.py /path/to/output_dir
``` 

## More examples

* [bop_scene_replication](../bop_scene_replication): Replicate the scenes and cameras from BOP datasets in simulation.
* [bop_object_physics_positioning](../bop_object_physics_positioning): Drop BOP objects on planes and randomize materials
* [bop_object_on_surface_sampling](../bop_object_on_surface_sampling): Sample upright poses on plane and randomize materials
