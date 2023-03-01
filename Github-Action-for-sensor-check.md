## Secrets required for Github Action
* Set GMAIL_USERNAME variable
> This variable is Sender's email account
* Set GMAIL_PASSWORD variable
> This variable is Sender's email password
* Set DOCKERHUB_USERNAME variable
> This variable is Dockerhub username
* Set DOCKERHUB_TOKEN variable
> This variable is Dockerhub password


### Steps in Github Actions
1. Set up rclone command by using config files from secrets
```yaml
      - name: Setup Rclone
        uses: AnimMouse/setup-rclone@v1
        with:
          rclone_config: ${{ secrets.RCLONE_CONFIG }}

```

2. Download R script from S3 bucket or Github
> From S3 bucket
```yaml
     - name: Download R script from S3
        run: rclone copy s3flare:wvwa-graphs/scripts/wvwa-generate-graphs.R ${{ github.workspace }}

```
> From Github
```yaml
      - name: Download R script from GitHub
        run: wget https://raw.githubusercontent.com/FLARE-forecast/FCRE-data/wvwa-generate-graphs/wvwa-generate-graphs.R

```

3. Log into Dockerhub with credentials set up in secrets
```yaml
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_PASSWORD }}

```

4. Run docker to do prediction
```yaml
     - name: Run automatic prediction file
        run:  | 
          docker run --rm -v ${{ github.workspace }}:/root/flare yjungku/wvwa-generate-graphs

```

5. Commit files to repo
```yaml
     - name: Commit files
        run: |
          git config --local user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git config --local user.name "github-actions[bot]"
          git add WeirDataFigures_$(date '+%Y-%m-%d').pdf MetDataFigures_$(date '+%Y-%m-%d').pdf BVRDataFigures_$(date '+%Y-%m-%d').pdf \
          FCRCatwalkDataFigures_$(date '+%Y-%m-%d').pdf CCRMetDataFigures_$(date '+%Y-%m-%d').pdf CCRWaterQualityDataFigures_$(date '+%Y-%m-%d').pdf
          git commit -m "wvwa-generate-graphs-$(date '+%Y-%m-%d')"
          git branch wvwa-graphs
          git push --force origin wvwa-graphs

```

6. Upload files to S3 bucket
```yaml
     - name: Copy files to s3 buckets
        run: |
          rclone copy ${{ github.workspace }}/WeirDataFigures_$(date '+%Y-%m-%d').pdf s3flare:wvwa-graphs/outputs/
          rclone copy ${{ github.workspace }}/MetDataFigures_$(date '+%Y-%m-%d').pdf s3flare:wvwa-graphs/outputs/
          rclone copy ${{ github.workspace }}/BVRDataFigures_$(date '+%Y-%m-%d').pdf s3flare:wvwa-graphs/outputs/
          rclone copy ${{ github.workspace }}/FCRCatwalkDataFigures_$(date '+%Y-%m-%d').pdf s3flare:wvwa-graphs/outputs/
          rclone copy ${{ github.workspace }}/CCRMetDataFigures_$(date '+%Y-%m-%d').pdf s3flare:wvwa-graphs/outputs/
          rclone copy ${{ github.workspace }}/CCRWaterQualityDataFigures_$(date '+%Y-%m-%d').pdf s3flare:wvwa-graphs/outputs/
```

7. Send email if there is a failure happened. Sender's email credentials is saved inside secrets.
```yaml
- name: Send mail
        if: ${{ failure() }}
        uses: dawidd6/action-send-mail@v2
        with:
          # mail server settings
          server_address: smtp.gmail.com
          server_port: 465
          # user credentials
          username: ${{ secrets.EMAIL_USERNAME }}
          password: ${{ secrets.EMAIL_PASSWORD }}
          # email subject
          subject: ${{ github.job }} job of ${{ github.repository }} has ${{ job.status }}
          # email body as text
          body: ${{ github.job }} job in worflow ${{ github.workflow }} of ${{ github.repository }} has ${{ job.status }}
          # comma-separated string, send email to
          to: 
          # from email name
          from: FLARE

```
> "to" variable is used to fill up with receipents' email address.
