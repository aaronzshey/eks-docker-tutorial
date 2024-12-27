# How to deploy a website to AWS EKS using Docker Desktop on Windows 11

Throughout this tutorial we'll use PowerShell commands.

I'm a big proponent of using package managers.  Opening MSI's and exe's, clicking install, and dragging and dropping to the right folder (or even worse, running $HOME or %USERPROFILE%) are tantamount to torture to me.  Even worse, having to do it all again after a new version is released?  Just kill me now. 

Here's how you can set up eksctl, the aws cli, Kubernetes, and more without having to click any download buttons (okay, only once).

## Download Docker Desktop

I swear it'll all be worth it! 

Go search on the internet for "Docker Desktop".  Download it and don't pay a single cent (the free tier works great).

After downloading and making a Docker account and then logging into the Docker Desktop app, click on the gear-shaped button on the upper right hand side, go to "Kubernetes" on the sidebar, and check in the box that says "enable Kubernetes". 

Great!  Now Docker Desktop will automatically keep your Kubernetes version up-to-date, and you don't have to lift a single finger.  Well, you do have to open Docker Desktop any time you want to do anything Kubernetes related but what's the big deal.

## Setting up eksctl

You can imagine my dismay that eksctl does NOT recommend using third-party installers - and when I did install eksctl via `winget`, it didn't even work.  

Thankfully, eksctl offers an easy and hassle-free usage method on their [AWS ECR page](https://gallery.ecr.aws/eksctl/eksctl) (Amazon Web Services Elastic Container Registry):

`alias eksctl='docker run --rm -it -v ~/.aws:/root/.aws -v $(pwd):/eksctl public.ecr.aws/eksctl/eksctl'`

Honestly who knows if this is all worth the effort.  Maybe eksctl is smart like Python and can upgrade itself after the first install.

Anyway, if you open your PowerShell profile (I have no idea where yours could be just google it) and add this:

``` bash
function e {
    docker run --rm -it -v $HOME/.aws:/root/.aws -v ${PWD}:/eksctl public.ecr.aws/eksctl/eksctl $args
}

Set-Alias -Name eksctl -Value e
```

You can restart your terminal, and then type `eksctl version` to see that eksctl now works! All thanks to Docker Desktop, you won't have to download another version of eksctl ever again.

Great!  Now all you need is the AWS command-line toolset.

## Installing AWS CLI

Just type `winget install --id "Amazon.AWSCLI"`

## Deploying

Okay, now we can get to the deployment part.

Make a new project folder and create a new file named `cluster-config.yaml` inside (I named mine aws-proj):

```
New-Item aws-proj -Type "directory"
cd aws-proj
New-Item cluster-config.yaml
```

Open `cluster-config.yaml` and paste this inside:

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: web-quickstart
  region: us-west-1

autoModeConfig:
  enabled: true
```

Windows 11 doesn't ship with vim or nano or any of that, so you can use `notepad cluster-config.yaml` to edit it.

Feel free to replace `us-west-1` with a region of your choice, like `us-east-1` if you don't live on the Best Coast.

Now type this into your terminal:

```
eksctl create cluster -f cluster-config.yaml
```

You'll see a bunch of output, hopefully nothing in red.  You'll eventually see some text like "waiting for cluster...".

If you're interested in seeing what's going on behind the scenes, you can log in to the [CloudFormation](console.aws.amazon.com/cloudformation) console to watch the activity logs.  However, you have to wait for your terminal on your computer to finish whatever it's doing, so looking at the console is mostly to pass the time.

Once that's done, it's time to use Kubernetes to actually start doing cool code engineer things.

A quick overview: Kubernetes is software that can deploy Docker images.  A Docker image is a fancy way to say a program that's been packaged into a container.  A container is a combined program and environment to run the program in.  For example, if we want to deploy a React website, a Docker container would contain instructions on how to build it (npm install, etc etc).

That explanation probably sucked that's why I'm unemployed.  

Anyway, it's time to get back to mindlessly copying and pasting commands.

You need to link your kubectl config to your EKS cluster - basically, Kubernetes needs to know where to deploy your sample application.  If you don't do this step, you can open Docker and see that whatever you are trying to deploy is currently running on your own machine.

Enter this command:

```
aws eks update-kubeconfig --region us-west-1 --name web-quickstart

```

As you know by now, replace the region with your region of choice, and replace name with the name you put into the previous config file.


Now, you can finish deploying like usual.  Copying the steps from [AWS' quickstart guide](https://docs.aws.amazon.com/eks/latest/userguide/quickstart.html):

create `ingressclass.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: alb
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: eks.amazonaws.com/alb
```

And run the following commands:

```
kubectl apply -f ingressclass.yaml
kubectl create namespace game-2048 --save-config
kubectl apply -n game-2048 -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.8.0/docs/examples/2048/2048_full.yaml
kubectl get ingress -n game-2048
```

Wait a little bit, and then visit the url displayed by the `get ingress` command.

That's it!  Hope it worked, if it didn't file an issue please :)