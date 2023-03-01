## Secrets required for Github Action
* Set GMAIL_USERNAME variable
> This variable is Sender's email account
* Set GMAIL_PASSWORD variable
> This variable is Sender's email password
* Set DOCKERHUB_USERNAME variable
> This variable is Dockerhub username
* Set DOCKERHUB_TOKEN variable
> This variable is Dockerhub password



Edit yaml file
---
#### The yaml file is used to include files attached in the email
##### *Take config-fcre.yml as example*

```yaml
gmail:
  username:
  password:
  from: SCC
```
> "username" and "password" variables will be added by secrets when executing Github action. "From" variable is used as sender's name.


```yaml
email:
  sender: SCC
  subject: Daily FCR Forecast and Graphs
  body: "Hello from Jetstream 2!\n\nHere are today's forecast and graphs. Have a wonderful day!\n\nBests,\nSCC and CIBR Team"
```

> "Subject" variable is used to be email subject.
> "Body" variable will be shown as the content inside the email.


```yaml
attachments_web:
    - prefix: https://github.com/FLARE-forecast/flare-dashboard/raw/main/docs/fcre_files/figure-html/bvre-ice-1.png
      infix:
      suffix:
```
> "attachments_web" variable is used to attach files from the web.
> 'Prefix', 'infix', and 'suffix' can be blank.

```yaml
attachments_local:
    - prefix:
      infix:
      suffix:
```
> "attachments_local" variable is used to attach files from local directories inside the container.
> 'Prefix', 'infix', and 'suffix' can be blank.

```yaml
recipients:
    - 
```
> "recipients" variable is used to fill in receivers' email address.

### Steps in Github Actions
1. Put gmail account name and password from secrets into yaml files
```yaml
      - name: Update gmail_username
        uses: fjogeleit/yaml-update-action@main
        with:
          valueFile: 'config-fcre.yml'
          propertyPath: gmail.username
          value: ${{ secrets.GMAIL_USERNAME }}
          commitChange: false
    
      - name: Update gmail_password
        uses: fjogeleit/yaml-update-action@main
        with:
          valueFile: 'config-fcre.yml'
          propertyPath: gmail.password
          value: ${{ secrets.GMAIL_PASSWORD }}
          commitChange: false

```
2. Log into Dockerhub by using credentials set up in secrets
```yaml
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

```
3. Run send-email container by using 'docker run'
```yaml
      - name: Run send-email container
        run: |
          docker run --rm -v ${{ github.workspace }}:/root/flare/config --env CONFIG_FILE="/root/flare/config/config-fcre.yml" yjungku/send-email:dev

```