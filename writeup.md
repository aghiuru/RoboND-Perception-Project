[clusters]: ./images/clusters.png
[confusion_matrix]: ./images/confusion_matrix.png
[labels-1]: ./images/labels-1.png
[labels-2]: ./images/labels-2.png
[labels-3]: ./images/labels-3.png
[pass_through]: ./images/pass_through.png
[point_cloud]: ./images/point_cloud.png
[ransac]: ./images/ransac.png
[voxel]: ./images/voxel.png

## Project: Perception Pick & Place
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


# Required Steps for a Passing Submission:
1. Extract features and train an SVM model on new objects (see `pick_list_*.yaml` in `/pr2_robot/config/` for the list of models you'll be trying to identify).
2. Write a ROS node and subscribe to `/pr2/world/points` topic. This topic contains noisy point cloud data that you must work with.
3. Use filtering and RANSAC plane fitting to isolate the objects of interest from the rest of the scene.
4. Apply Euclidean clustering to create separate clusters for individual items.
5. Perform object recognition on these objects and assign them labels (markers in RViz).
6. Calculate the centroid (average in x, y and z) of the set of points belonging to that each object.
7. Create ROS messages containing the details of each object (name, pick_pose, etc.) and write these messages out to `.yaml` files, one for each of the 3 scenarios (`test1-3.world` in `/pr2_robot/worlds/`).  [See the example `output.yaml` for details on what the output should look like.](https://github.com/udacity/RoboND-Perception-Project/blob/master/pr2_robot/config/output.yaml)
8. Submit a link to your GitHub repo for the project or the Python code for your perception pipeline and your output `.yaml` files (3 `.yaml` files, one for each test world).  You must have correctly identified 100% of objects from `pick_list_1.yaml` for `test1.world`, 80% of items from `pick_list_2.yaml` for `test2.world` and 75% of items from `pick_list_3.yaml` in `test3.world`.
9. Congratulations!  Your Done!

# Extra Challenges: Complete the Pick & Place
7. To create a collision map, publish a point cloud to the `/pr2/3d_map/points` topic and make sure you change the `point_cloud_topic` to `/pr2/3d_map/points` in `sensors.yaml` in the `/pr2_robot/config/` directory. This topic is read by Moveit!, which uses this point cloud input to generate a collision map, allowing the robot to plan its trajectory.  Keep in mind that later when you go to pick up an object, you must first remove it from this point cloud so it is removed from the collision map!
8. Rotate the robot to generate collision map of table sides. This can be accomplished by publishing joint angle value(in radians) to `/pr2/world_joint_controller/command`
9. Rotate the robot back to its original state.
10. Create a ROS Client for the “pick_place_routine” rosservice.  In the required steps above, you already created the messages you need to use this service. Checkout the [PickPlace.srv](https://github.com/udacity/RoboND-Perception-Project/tree/master/pr2_robot/srv) file to find out what arguments you must pass to this service.
11. If everything was done correctly, when you pass the appropriate messages to the `pick_place_routine` service, the selected arm will perform pick and place operation and display trajectory in the RViz window
12. Place all the objects from your pick list in their respective dropoff box and you have completed the challenge!
13. Looking for a bigger challenge?  Load up the `challenge.world` scenario and see if you can get your perception pipeline working there!

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.

You're reading it!

### Exercise 1, 2 and 3 pipeline implemented
#### 1. Complete Exercise 1 steps. Pipeline for filtering and RANSAC plane fitting implemented.

Note: I have not included any pictures for the exercises. I saved the point cloud of the first world (in a file) for the project and included pictures of the same perception pipeline (it's almost the same code after all) processing that point cloud instead. I hope that's fine.

Filled out the TODOs according to instructions and adjusted required parameters until the clusters matched the shape of the objects in the point cloud. I got there by trial and error -- sometimes I would cut a slice parallel to the table plane that was too wide, sometimes not enough.

#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.

Not that much to comment here. Again, got to the final values of the parameters by trial and error.

#### 3. Complete Exercise 3 Steps.  Features extracted and SVM trained.  Object recognition implemented.

Not much to comment here either. Initially I thought it took a while to capture the features, but then it took way longer to capture the features for the project. :)

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

##### 1.1. Perception pipeline

First step I took was to load the first world and save the point cloud detected by the camera to a file.
This way I could progress faster since the project envirnoment takes up a lot of resources while running.

![point cloud][point_cloud]

Then I went on by implementing the statistical outlier filter to get rid of the camera noise.

On top of that I applied the voxel grid filter that reduced the original point cloud size significantly.
You could tell that by the time it took to open the filtered version.

![voxel grid][voxel]

Next step was to get rid of the groud and table.
That was achieved by applying a passthrough filter and plane detection.
Parameters had to be adjusted to the new height of the table.
Also I had to crop the dropboxes that showed up on the sides out of the frame.

![pass through][pass_through]

![after removing the table][ransac]

The final step of the perception pipeline was clustering the points left into separate objects.

![clusters][clusters]


##### 1.2. Object detection

For the detection, I had first to generate features for all objects that could possibly be part of a world and then, based on those features, train an SVM that would do the actual detection.

For the number of features generated, I chose 50. Even though the confusion matrix shows number in the 80s range, it proved to be enough for all test worlds. I would have love to generate more, as sometimes objects get misclassified, but on my computer Rviz runs very slow. Maybe I'll put the amazon credit to work and rent a spot instance for the last project. On to the confusion matrix!

![confusion matrix][confusion_matrix]

Below are the screenshots with labels for the three worlds.

![world 1][labels-1]
![world 2][labels-2]
![world 3][labels-3]
