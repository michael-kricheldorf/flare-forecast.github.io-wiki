1- [Create a Personal Access Token in GitHub.](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

2- [Create an encrypted secret for the repository.](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- The secret can be named as you wish. In our example, we called it `PAT`.

- Copy the token from the previous step and use it as the value of the secret.

3- Add the secret as a build argument in the YAML workflow file for your workflow step.
- You can name the build argument as you wish. We named it `GITHUB_PAT` in our sample:
```yml
      - name: Build and push
        uses: docker/build-push-action@v3
        with:
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/flare-rocker:4.2.2
          build-args: GITHUB_PAT=${{ secrets.PAT }}
```

4- Read the build argument in Dockerfile as an argument and pass it to an environment variable.
- You can technically name the environment variable as you wish, but since some functions such as `install_github()` use `GITHUB_PAT` environment variable as their default token for authentication, it is highly recommended that you name it `GITHUB_PAT`. In a case, naming it anything than `GITHUB_PAT` results in "API rate limit error", caused by unauthenticated calls to install dependencies and updates by `install_github()` despite using the environment variable as the authentication token.

```Dockerfile
ARG GITHUB_PAT
ENV GITHUB_PAT=$GITHUB_PAT
```

Then it will be automatically used for installing packages using `install_github()`:

```Dockerfile
RUN R -e "devtools::install_github('FLARE-forecast/FLAREr')"
```

Or you can explicitly use it:

```Dockerfile
RUN R -e "devtools::install_github('FLARE-forecast/FLAREr', auth_token = $GITHUB_PAT)"
```