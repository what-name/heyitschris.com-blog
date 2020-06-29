![Deployment](https://github.com/what-name/heyitschris.com-blog/workflows/Deploy%20to%20S3%20bucket/badge.svg)

## Info
This repository contains my personal blog's Hugo project.
[blog.heyitschris.com](https://blog.heyitschris.com)

## Deploy
```bash
git clone https://github.com/what-name/heyitschris.com-blog
cd heyitschris.com-blog
hugo
# edit config.toml file under distribution
hugo deploy
```

## CI/CD
As detailed in the `.github/workflows/main.yml` file, this repository utilizes a GitHub Actions pipeline to build and deploy my blog on push activity to the main branch. It authorizes against AWS with the use of GitHub Secrets, builds the Hugo project, deploys the new files to the S3 bucket and finally invalidates the CloudFront cache. The reason for the forced invalidation is that by skipping that step, the blog's front page will be inconsistent when publishing new posts.