# Sample Code For Preview Environments With Argo CD, from a Helm repository
# Based heavily on
* [Environments Based On Pull Requests (PRs): Using Argo CD To Apply GitOps Principles On Previews](https://youtu.be/cpAaI8p4R60)


# Demo

Retrieve the latest version of a chart from a chart repo and then deploy a slightly different version of it for fun.

# Making this work

#### Install argocd and get it running(?)

```
brew install argocd
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
argocd login localhost
```

hopefully `localhost:8080` now runs argo cd!

#### We're going to create apps with `kubectl` so that they are _as code_ but this is only one way to skin this cat. You can also use   `argocd app create --repo https://github.c....` pointed at this repo. We'll do that in the future.

```
 kubectl apply --filename project.yaml
 kubectl apply --filename apps.yaml
```

Creates an `argocd` project and an `argocd` app. This is your app of apps and it is called 'previews'.

##### APP OF apps?

https://argoproj.github.io/argo-cd/operator-manual/cluster-bootstrapping/ This is so you can create an argo map to manage a set of apps. So the `previews` app will contain and manage all the preview environments. This will work with `GitOps` which will be automated by `PLATFORM-103`.


###### How do I sync a new app?


Hardcoded in `helm/templates/pr-nginx-1.yaml` + 2 are some pre-hardcoded files that create `nginx` applications in `kubernetes`, they have slightly different things, so one is on `NodePort` `30010` and one on `30011`, and they run different `nginx` containers.

You can probably hit `localhost:30001` to see an nginx app, if not, then make sure `argocd` is synced.

You can run 
```
cp helm/templates/pr-nginx-1.yaml helm/templates/pr-nginx-3.yaml
gsed -i s/pr-nginx-1/pr-nginx-3/g  helm/templates/pr-nginx-3.yaml && gsed -i s/30001/30012/g  helm/templates/pr-nginx-3.yaml 
git add helm/templates/pr-nginx-3.yaml
git commit helm/templates/pr-nginx-3.yaml -m  'create something new'
git push
```



You can probably hit `localhost:30003` to see a _NEW_ nginx app, if not, then make sure `argocd` is synced.


---

### GitHub actions:

The steps above (creating the extra file and replacing the contents before pushing it to git) can be done in `PLATFORM-103` in GitHub actions to `cutover-core`, there are tools like `kyml` to help, and probably better ways to do it..

We'd need to pass the `branch name` `commit` `PR id` and more in the `GitHub` action

Then we'd want to remove the file when the `PR` is closed.
