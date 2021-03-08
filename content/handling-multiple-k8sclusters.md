# Handling multiple (local) Kubernetes clusters

When you start working with Kubernetes, you will sooner or later end up with multiple k8s clusters that you need to interact with. Promise. The good news is, kubectl, the Kubernetes Command Line Interface (CLI) client, has built-in support for multiple clusters. There is however a couple of things you should know, a number of traps that you might want to avoid, and a number of tools you will want to adopt early on in your Kubernetes journey. Investing time to wrap your head around these things will save you a lot of time and hassle. 

Now, this is certainly not a new topic, and all of this info can be found somewhere else. But it is spread out in a number of places, which makes it a bit challenging to figure things out - particularly when you're new to the Kubernetes world. So collecting (hopefully) all relevant information in one place and putting it into a logical order seemed useful.  

## Kubernetes configuration files and context switching

Before we start discussing multiple clusters, let's take a brief look at how kubectl finds out how to talk to a particular Kubernetes cluster. By the way, if you're looking for a general introduction to kubectl, have a look at [this tutorial](https://rancher.com/learning-paths/how-to-manage-kubernetes-with-kubectl/) by Kelly Griffin. 

The most widely used mechanism to find cluster access information re so-called _kubeconfig files_. By default, kubectl will use information from the file `$HOME/.kube/config`. It is possible to point to other config files by setting the `KUBECONFIG` environment variable or by using the `--kubeconfig` flag. There are three components in a kubeconfig file:
* clusters: describes your - you guessed it - clusters, i.e. specifies name, API server URL and certificate authority. 
* users: the user credentials needed to authenticate to your cluster(s), i.e. name and either key pairs or username/password pairs
* context: a combination of cluster, user and namespace to exactly define the target for your subsequent kubectl operations. Even if you only have one cluster, you can either have multiple users or different namespaces to work with, so the context construct makes it easier to switch between your most frequently used combinations. 

You can of course just manually edit your kubeconfig files, but you can also manipulate them with a number of `kubectl config` subcommands:
* `kubectl config --kubeconfig=config-demo set-cluster` to define clusters
* `kubectl config --kubeconfig=config-demo set-credentials` for setting user credentials
* `kubectl config --kubeconfig=config-demo set-context` to define or change contexts

Once you have set up your clusters, users and contexts, you can easily switch between contexts using the `kubectl config use-context` command. The currently active context can be checked with `kubectl config current-context`. 

A more detailed description of all this can be found at [https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/]. 

Theoretically, this could be the end of the article, if it wasn't for the fact that once the number of clusters you interact with grows beyond low single-digit numbers, keeping all information in one config file becomes, let's say, unpleasant. 

## Multiple config files

A better way is to keep information about different clusters in different config files. There's basically two ways to handle multiple config files: the simple way and the clever way. The simple way is you manually specify the config you want to use each time you call kubectl via the `--kubeconfig` flag. The clever way is to concatenate multiple kubeconfigs into your `KUBECONFIG` environment variable. On Linux and Mac, the list is delimited by a colon, while for Windows you'll use a semicolon. 

The rules according to which kubectl identifies its valid configuration data from this list of files are described in detail in the [Kubernetes documentation]{https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/}, but here's the 10000 m summary:
* Files specified via --kubeconfig always win, no matter what's in your 
* The content of all configs specified in $KUBECONFIG is merged
* The first file to set a value or map key wins
* Never change a value or map key. Even if a later occurrence has non-conflicting values, discard that info
* The first context found in the chain wins - if you have a context with the same name in a later file, kubectl will never find it. 
TODO: check if this is complete

You can use `kubectl config view` to check what this leads to on your system.

TODO: talk about manually merging multiple files into one? 
kubectl config view --merge --flatten

TODO: Talk about extracing one context into a single file?
kubectl config view --minify --flatten --context=my_context > myconfig.yaml

## Auto-populate $KUBECONFIG

how to set up $KUBECONFIG so that it automatically picks up all files from ~/.kube/

## Useful tools

So far we've obviously only covered Kubernetes' built-in functionality. Which is basically fine, but the are more convenient ways to do things. 

The first tool that I want to call out is Ahmet Balkan's kubectx and its brother-in-arms, kubens. Both are available from the same [GitHub repo](https://github.com/ahmetb). There's really not much to explain about them - they give you a shorter way to switch between contexts and namespaces than their somewhat lengthy kubectl counterparts, plus they basically behave like the cd command for changing directories (e.g. `kubectx -` gets you back to the previous context). Nothing magic, but neat. Check the embedded videos in the GitHub readme, and you'll get the idea. 

Another widely used tool is kube-ps1, which adds information on cluster and namespace from your current context to your command prompt. You might think this is unnecessary, but wait until you accidentally delete a pod from the wrong namespace. You can switch between your normal prompt and the one enriched by kube-ps1 with the `kubeon` command and turn it back off with `kubeoff`. 
Tools to show current config & namespace on prompt?
https://github.com/jonmosco/kube-ps1 

direnv to set KUBECONFIG based on current directory tree. Useful to tie local development to a particular namespace. 

Resources: 
https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/
https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/ 

Talk about k9s, KUI?

Talk about version mismatch in practice, multiple versions of kubectl?

TODO: add a CTA to check out other resources and join the community. 