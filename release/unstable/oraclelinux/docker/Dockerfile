# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.
#
# Docker image file that describes an Oracle Linux image with PowerShell
# installed from RHEL7 PowerShell package

# Define arg(s) needed for the From statement
FROM oraclelinux:8.10 AS installer-env

# Define Args for the needed to add the package
ARG PS_VERSION=7.0.1
ARG PS_PACKAGE=powershell-${PS_VERSION}-1.rh.x86_64.rpm
ARG PS_PACKAGE_URL=https://github.com/PowerShell/PowerShell/releases/download/v${PS_VERSION}/${PS_PACKAGE}
ARG PS_INSTALL_VERSION=7

# Define Args and Env needed to create links
ENV \
    PS_INSTALL_FOLDER=/opt/microsoft/powershell/$PS_INSTALL_VERSION \
    # Define ENVs for Localization/Globalization
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    # Set up PowerShell module analysis cache path
    PSModuleAnalysisCachePath=/var/cache/microsoft/powershell/PSModuleAnalysisCache/ModuleAnalysisCache  \
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-OracleLinux-8.3

# Installation
RUN \
    # update package list
    yum update -y \
    # install dependencies
    && yum install -y \
      # required for help in powershell
	    less \
      # required for SSH
      openssh-clients \
      # install EPEL repository
	    oracle-epel-release-el8 \
    # update package list from EPEL repository
    && yum update \
    # install dependencies from EPEL repository
    && yum install -y \
      # install gssntlmssp package
	    gssntlmssp \
    # download the powershell package
    && curl -sSL ${PS_PACKAGE_URL} -o /tmp/powershell-linux.rpm \
    # install powershell package
    && yum install -y /tmp/powershell-linux.rpm \
    # remove powershell package
    && rm -f /tmp/powershell-linux.rpm \
    # intialize powershell module cache
    # and disable telemetry
    && export POWERSHELL_TELEMETRY_OPTOUT=1 \
    && pwsh \
        -NoLogo \
        -NoProfile \
        -Command " \
          \$ErrorActionPreference = 'Stop' ; \
          \$ProgressPreference = 'SilentlyContinue' ; \
          while(!(Test-Path -Path \$env:PSModuleAnalysisCachePath)) {  \
            Write-Host "'Waiting for $env:PSModuleAnalysisCachePath'" ; \
            Start-Sleep -Seconds 6 ; \
          }" \
    # upgrade packages
    && yum upgrade -y \
    # clean cached data
    && yum clean all \
    # remove cache folders and files
    && rm -rf /var/cache/yum

CMD [ "pwsh" ]
