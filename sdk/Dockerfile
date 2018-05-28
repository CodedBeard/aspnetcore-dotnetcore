# escape=`

# Installer image
FROM microsoft/dotnet-framework:4.7.1-windowsservercore-ltsc2016 AS installer-env

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Retrieve .NET Core SDK
ENV DOTNET_SDK_VERSION 2.1.300

# install .NET Core SDK and VSBuild tools
RUN Invoke-WebRequest -OutFile dotnet.zip https://dotnetcli.blob.core.windows.net/dotnet/Sdk/$Env:DOTNET_SDK_VERSION/dotnet-sdk-$Env:DOTNET_SDK_VERSION-win-x64.zip; `
    $dotnet_sha512 = '4aa6ff6aa51e1d71733944e10fd9e37647a58df7efbc76f432b8c3ffa3f617f9da36f72532175a1e765dbaf4598a14350017342d5f776dfe8e25d5049696d003'; `
    if ((Get-FileHash dotnet.zip -Algorithm sha512).Hash -ne $dotnet_sha512) { `
        Write-Host 'CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    `
    Expand-Archive dotnet.zip -DestinationPath $Env:ProgramFiles\dotnet; `
    Remove-Item -Force dotnet.zip; `
    write-output 'downloading vsbuild'; `
    Invoke-WebRequest -UseBasicParsing https://download.visualstudio.microsoft.com/download/pr/12017762/e64d79b40219aea618ce2fe10ebd5f0d/vs_BuildTools.exe -outfile vsbuild.exe; `
    write-output 'installing vsbuild'; `
    Start-Process -FilePath C:\vsbuild.exe -ArgumentList '--quiet','--norestart','--add Microsoft.VisualStudio.Workload.WebBuildTools','--includeRecommended','--includeOptional' -Wait; `
    write-output 'removing vsbuild installer'; `
    Remove-Item -Force vsbuild.exe; 

RUN setx /M PATH $($Env:PATH + ';' + $Env:ProgramFiles + '\dotnet')

# Configure Kestrel web server to bind to port 80 when present
ENV ASPNETCORE_URLS=http://+:80 `
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true `
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true `
    # Skip extraction of XML docs - generally not useful within an image/container - helps perfomance
    NUGET_XMLDOC_MODE=skip

# Trigger first run experience by running arbitrary cmd to populate local package cache
RUN dotnet help

# Ensure packages required for standalone apps are pre-fetched
RUN mkdir $env:TEMP\warmup; `
    cd $env:TEMP\warmup; `
    dotnet new web --no-restore; `
    dotnet restore -p:RuntimeIdentifier=win7-x64 `
        --source https://api.nuget.org/v3/index.json --source https://dotnetfeed.blob.core.windows.net/orchestrated-release-2-1/20180515-07/final/index.json; `
    cd ..; `
rm -Recurse -Force $env:TEMP\warmup