FROM public.ecr.aws/codebuild/amazonlinux2-x86_64-standard:5.0
RUN command -v gh >/dev/null 2>&1 || (yum install -y yum-utils && yum-config-manager --add-repo https://cli.github.com/packages/rpm/gh-cli.repo && yum install -y gh)
RUN pip install copier taskcat yq jq awscliv2 cfn-flip
