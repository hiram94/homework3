# Homework3 - Generative Models 
This is a team homework.<br>
In your report please list the contribution of each team member.

# Task
Your task is to modify DCGAN-tensorflow <a>https://github.com/carpedm20/DCGAN-tensorflow</a> so that it can generate images of higher resolution (256x256).

You have to submit 500 images generated by your generator. The resolution of each image is 256x256 pixels.
We will collect the images from all teams and create an evaluation dataset containing those synthetic images and some real images.

You need to run your discriminator on this evaluation dataset. We will rank each team according to the prediction results. You also have to submit a report describing your modifications and references.

# Dataset for training
The training data are from Places2 standard dataset (small images 256x256)
<a>http://places2.csail.mit.edu/challenge2016/train_256_places365standard.tar</a>

We only use images of the following classes (indoor scenes):
<b>bedroom, childs room, dining room, dorm room, hotel room, living room, recreation room, storage room, television room, and waiting room.</b>
Just for your convenience, here is a compilation of images from those clasees:<br>
<a>https://dl.dropboxusercontent.com/u/26848284/indoor.tgz</a><br>
The link will be removed after homework due date.

# Important dates
* The generated images must be submitted before 6pm, ~~December 21~~ December 26.
* Evaluation dataset will be avaialbe after 10pm, ~~December 21~~ December 26.
* The prediction results must be submitted before 10am, ~~December 22~~ December 27.
* Your written report is due 10pm, ~~December 22~~ December 27.
