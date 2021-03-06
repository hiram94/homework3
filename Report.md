# Report

In this homework, I modified 3 files so that the model can generate images with 256x256 resolution and 2 files in order that the model can discriminate whether the input image is natural or artificial. For better clarity, I will demonstrate them separately.

##Modification for higher resolution

### 1. main.py

By repacing 64 with 256, the input images can have a resolution 256x256.

    flags.DEFINE_integer("output_size", 256, "The size of the output images to produce [256]")

### 2. model.py

Turning the output_size into 256 to output 256x256-resolusion images. Lowering the gf_dim as well as df_dim to make it easier for 2 appended layers.

    batch_size=64, sample_size = 64, output_size=256,
    y_dim=None, z_dim=100, gf_dim=16, df_dim=16,

Batch normalization for 2 appended layers.

    self.d_bn3 = batch_norm(name='d_bn3')
    self.d_bn4 = batch_norm(name='d_bn4')

    self.d_bn5 = batch_norm(name='d_bn5')

    self.g_bn3 = batch_norm(name='g_bn3')
    self.g_bn4 = batch_norm(name='g_bn4')

    self.g_bn5 = batch_norm(name='g_bn5')

Appending 2 layers for discriminator so that the image parameters can be reshaped smoothly.

    h4 = lrelu(self.d_bn4(conv2d(h3, self.df_dim*16, name='d_h4_conv')))
    h5 = lrelu(self.d_bn5(conv2d(h4, self.df_dim*32, name='d_h5_conv')))
    h6 = linear(tf.reshape(h5, [self.batch_size, -1]), 1, 'd_h5_lin')

    return tf.nn.sigmoid(h6), h6

Similarly, there are 2 appended layers for generator. However, the parameters for the original layers also need to be modified.

    s2, s4, s8, s16, s32, s64 = int(s/2), int(s/4), int(s/8), int(s/16), int(s/32), int(s/64)

    self.z_, self.h0_w, self.h0_b = linear(z, self.gf_dim*32*s64*s64, 'g_h0_lin', with_w=True)

    self.h0 = tf.reshape(self.z_, [-1, s64, s64, self.gf_dim * 32])

    self.h1, self.h1_w, self.h1_b = deconv2d(h0,
    [self.batch_size, s32, s32, self.gf_dim*16], name='g_h1', with_w=True)

    h2, self.h2_w, self.h2_b = deconv2d(h1,
    [self.batch_size, s16, s16, self.gf_dim*8], name='g_h2', with_w=True)

    h3, self.h3_w, self.h3_b = deconv2d(h2,
    [self.batch_size, s8, s8, self.gf_dim*4], name='g_h3', with_w=True)

    h4, self.h4_w, self.h4_b = deconv2d(h3,
    [self.batch_size, s4, s4, self.gf_dim*2], name='g_h4', with_w=True)
    h4 = tf.nn.relu(self.g_bn4(h4))

    h5, self.h5_w, self.h5_b = deconv2d(h4,
    [self.batch_size, s2, s2, self.gf_dim*1], name='g_h5', with_w=True)
    h5 = tf.nn.relu(self.g_bn5(h5))

    h6, self.h6_w, self.h6_b = deconv2d(h5,
    [self.batch_size, s, s, self.c_dim], name='g_h6', with_w=True)

    return tf.nn.tanh(h6)

Similarly, 2 appended layers are adapted to sampler. The parameters of every layer are given in proportion.

    s2, s4, s8, s16, s32, s64 = int(s/2), int(s/4), int(s/8), int(s/16), int(s/32), int(s/64)

    h0 = tf.reshape(linear(z, self.gf_dim*32*s64*s64, 'g_h0_lin'),
    [-1, s64, s64, self.gf_dim * 32])

    h1 = deconv2d(h0, [self.batch_size, s32, s32, self.gf_dim*16], name='g_h1')

    h2 = deconv2d(h1, [self.batch_size, s16, s16, self.gf_dim*8], name='g_h2')

    h3 = deconv2d(h2, [self.batch_size, s8, s8, self.gf_dim*4], name='g_h3')

    h4 = deconv2d(h3, [self.batch_size, s4, s4, self.gf_dim*2], name='g_h4')
    h4 = tf.nn.relu(self.g_bn4(h4, train=False))

    h5 = deconv2d(h4, [self.batch_size, s2, s2, self.gf_dim*1], name='g_h5')
    h5 = tf.nn.relu(self.g_bn5(h5, train=False))

    h6 = deconv2d(h5, [self.batch_size, s, s, self.c_dim], name='g_h6')

    return tf.nn.tanh(h6)

### 3. utils.py

This may be redundant. Just to make conditions for images with higher resolution.

    def get_image(image_path, image_size, is_crop=True, resize_w=256, is_grayscale = False):

To achieve the goal that during the testing process every output is a single image instead of a combined image with 64 sub-images.

    #for idx, image in enumerate(images):
    #i = idx % size[1]
    #j = idx // size[1]
    #img[j*h:j*h+h, i*w:i*w+w, :] = image
    img = images

Since npx represents the number of pixels of the width/height of image.

    def transform(image, npx=256, is_crop=True, resize_w=64):

The output will be 64 single images other than one image with 64 sub-images.

    for i in xrange(len(samples)):
    save_images(samples[i], [1, 1], './samples/test_arange_%s_%s.png' % (idx, i))


I also tried to replace the value of z_dim, 100, with 500 to generate 500 totally dissimilar images, however, it went wrong after training 499 images.

##Modification for discrimination

Thanks to my modification, we can now conveniently discriminate images by:

    $ python main.py --dataset DATASET_NAME --is_test True

### 1. main.py

With a new boolean function, we can start the discrimination work without affecting the original training and testing instructions of the model.

    flags.DEFINE_boolean("is_test", False, "True for discriminating, False for testing [False]")

This is to decide whether the model have to do testing or dicriminating work.

    if FLAGS.is_test:
        dcgan.test(FLAGS)
    else:
        # Below is codes for visualization
        OPTION = 1
        visualize(sess, dcgan, FLAGS, OPTION)
        
### 2. model.py

This is the detailed function for the discrimination work. It first restore the model parameters from checkpoint, then read the images according to the .txt file and finally predict the discrimination result and save the result to a new .txt file.

    def test(self, config):
        root = '.'
        Modelparameters = os.path.join(root, 'checkpoint')
        model_dir = 'DATASET_NAME_64_256'
        Modelparameters = os.path.join(Modelparameters, model_dir)
        model = tf.train.latest_checkpoint(Modelparameters)
        self.saver.restore(self.sess,model)
        print(" [*] Success to restore model")
        data = []
        evalFile = open('./HW3eval/eval.txt','r')
        Filenames = evalFile.readlines()
        for line in Filenames:
            try:
                img = transform(imread('./HW3eval/eval/'+line[:-1]+'.png'), 122, False)
            except:
                img = transform(imread('./HW3eval/eval/'+line[:-1]+'.jpg'), 122, False)
            data.append(img)
        print(" [*] Success to read data")
        Resultfile = open('test_result.txt','w')
        Iterations = len(data)//self.batch_size +1
        for j in xrange(Iterations):
            print(" [*] %d" % j)
            sample = data[j:j+self.batch_size]
            D, D_logits = self.sess.run([self.D, self.D_logits], feed_dict = {self.images:sample})
            for k in xrange(len(D)):
                if D[k] [0]>0.5:
                    Result = 1
                else:
                    Result = 0
                Resultfile.write('%d\n'%(Result))
        Resultfile.close()
