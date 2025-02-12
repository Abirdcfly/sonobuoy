---
name: Weekly release
about: Ensure weekly release gets done and follows documentation
title: 'Weekly release'
labels: release

---

## Confirming latest k8s compatibility

 - Is there a Kubernetes release since the last Sonobuoy release? If so, apply the following steps:
 - [ ] Ensure the upstream conformance script is working appropriately:
   - Update the `kind-config.yaml` file with the new image version [here](https://github.com/vmware-tanzu/sonobuoy/blob/main/kind-config.yaml).
   - Run quick mode to confirm sonobuoy/conformance test work properly with this k8s version
   - Run `sonobuoy images` and get a list of test images 
 - [ ] Update the data supporting the e2e command (test lists). The steps are outlined [here](https://sonobuoy.io/docs/dryrun-listgenerator/). You can do this apart from the release process as well.

## Docs and versioning

 - [ ] Updating the versioned docs
   - Explicit doc changes (if any) should be made to the appropriate files in directory `site/docs/main`.
   - Next, generate a set of versioned docs for v0.x.y. For instance, the new docs be generated by running the command:

```
./scripts/update_docs.sh -b -v v0.x.y
```

This will copy the current `main` docs into the version given and update
a few of the links in the README to be correct. It will also update
the website config to add the new version and consider it the newest
version of the docs. The `-b` flag also bumps the version in the code to match.

## PR and Tag

- [ ] Create PR
  - Commit previous changes and open a new PR
  - Ensure your commits signed
  - Follow workflow progress in GithubActions. Once all checks passes, merge

- [ ] Tag release
  - This step will tag the code and triggers a release.
  - Pull the latest `main` to include the PR you just merged. Then, on the `main` branch, create an annotated `tag` for the commit that was merged and push it to upstream repo.

```
git tag -a v0.x.y -m "Release v0.x.y"
git push upstream refs/tags/v0.x.y
```

If there is a problem and you need to remove the tag, run the following commands

 ```
 git tag -d v0.x.y
 git push upstream :refs/tags/v0.x.y
 ```

 > NOTE: The `:` preceding the tag ref is necessary to delete the tag from the remote repository.
 > Git refspecs have the format `<+><src>:<dst>`.
 > By pushing an empty `src` to the remote `dst`, it makes the destination ref empty, effectively deleting it.
 > For more details, see the [`git push` documentation](https://git-scm.com/docs/git-push) or [this concise explanation on Stack Overflow](https://stackoverflow.com/a/7303710).


## Release Validation
 - [ ] Open a browser tab and go to: https://https://github.com/vmware-tanzu/sonobuoy/actions and verify go releaser for tag v0.x.y completes successfully.
 - [ ] Upon successful completion of build job above, check the [releases tab of Sonobuoy](https://github.com/vmware-tanzu/sonobuoy/releases) and verify the artifacts and changelog were published correctly.
 - [ ] Run the following command to make sure the image was pushed correctly to [Docker Hub](https://cloud.docker.com/u/sonobuoy/repository/docker/sonobuoy/sonobuoy/tags):

```
docker run -it sonobuoy/sonobuoy:v0.x.y /sonobuoy version
```

The `Sonobuoy Version` in the output should match the release tag above.
 - [ ] Go to the [GitHub release page](https://github.com/vmware-tanzu/sonobuoy/releases) and download the release binaries and make sure the version matches the expected values.
 - [ ] Run a [Kind](https://github.com/kubernetes-sigs/kind) cluster locally and ensure that you can run `sonobuoy run --mode quick`.

Once the release is done, we copy the images over to our Harbor mirror. This is to provide a non-rate-limited point of access. Simply run:

```
# Ensure you are connected to the VPN when you run this, otherwise you will get an auth error even if login credentials are correct.
skopeo copy --all "docker://registry.hub.docker.com/sonobuoy/sonobuoy:v0.56.1" "docker://projects.registry.vmware.com/sonobuoy/sonobuoy:v0.56.1"
```

## Release notes

 - [ ] Update the release notes if desired on GitHub by editing the newly created release. We publish more thorough details in a blog post on a regular basis, so only minor updates to this should be required.
