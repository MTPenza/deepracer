### Setup
- Navigate to [CloudShell](https://us-east-1.console.aws.amazon.com/cloudshell/home?region=us-east-1#9260f248-1a72-48d9-b7b2-48b4eb8e7708)
- `cd pit-crew-wizards/deepracer-on-the-spot`
- Adjust custom-files/* as wanted

### Copy pre-existing models to S3 (if wanted)
- Navigate in your browser to: AWS DeepRacer -> Your models -> <specific_model>
- Select the "Actions" tab on the right
- "Copy to S3"
    - Create a new bucket
- Run the following command with your own substitutions in Cloudshell:
        -  `aws s3 cp "s3://<bucketCreatedInPreviousStep>/<modelName>/<date>/" s3://pcw-deepracer-base-bucket-um12ndpazfns/<modelName> --recursive`
- Store model and logs
    - One can store video too, but more expensive and no point

### Training
- `curl -s checkip.dyndns.org`
    - IP might change periodically
- `./create-base-resources.sh pcw-deepracer-base <ip_address>`
    - Should only need to be done one time, but doesn't matter if run again
- Open `custom-files/run.env`
- Set `DR_LOCAL_S3_MODEL_PREFIX` to the new model name (eg, "PCW-PPL-VV-12")
- `DR_LOCAL_S3_PRETRAINED_PREFIX` to the model you want to clone/train based on
    - If not cloning/basing off model, then instead set `DR_LOCAL_S3_PRETRAINED=False`
- `./create-spot-instance.sh pcw-deepracer-base pcw-<training-instance_name> 30` OR `./create-standard-instance.sh pcw-deepracer-base <training_instance_name> 30`
    - 30 = time to live. Can change as wanted
    - Alternative command: `./create-standard-instance.sh`. Spot instances cost less but also offer less availability
        - I (styx) have very rarely been able to get a spot instance
    - `training_instance_name` gives a name for current training. Doesn't matter much but should change for each training
- Check [EC2 instances](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Instances:v=3;$case=tags:true%5C,client:false;$regex=tags:false%5C,client:false) after a few minutes - you should see one pop up
    - If creating non-spot instances, ensure the instances our script creates are terminated. This process should happen automatically
- Browse to the IP the `create-*-instance` command outputs. If you fail to connect to the IP you may need to modify security rules
    - Modify inbound rules to allow all ipv4 traffic


### Known issues

#### Impermanent security rules
- Problem: Rules added to security group work at first, then mysteriously delete after a time
- Problem significance:
    - SSH cannot reliably function
        - Workaround: Use CloudShell instead of SSH
    - Accessing training webview can sporadically fail
        - Workaround: Must update security rule every time the rule deletes itself

##### Solution
- Specify your home IP address


#### Track name differs from internet research
- Internet research: https://github.com/aws-deepracer-community/deepracer-log-guru/blob/master/src/tracks/ace_speedway_ccw_track.py
    - 2022_april_open_ccw = ace speedway counterclockwise
- Uploading to Amazon produces world of reinvent:2018 instead of ace speedway
##### Solution
- Track will always display as reinvent:2018 after importing no matter what. Don't worry about it

### Goal (seconds)
- City finals: 25 seconds max
- World: 20 seconds max

