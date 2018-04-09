# escape=`

FROM codedbeard/aspnetcore-servercore

# set up environment
ENV ASPNETCORE_URLS http://+:80

# Retrieve node and git
## https://nodejs.org/dist/v8.9.4/
ENV NODE_VERSION 8.9.4
ENV NODE_DOWNLOAD_SHA 48946e99ac4484e071df25741d2300f3a656f476c5ff3f8116a4746c07ebe3b7
RUN Invoke-WebRequest -UseBasicParsing https://nodejs.org/dist/v${env:NODE_VERSION}/node-v${env:NODE_VERSION}-win-x64.zip -outfile node.zip; `
    if ((Get-FileHash node.zip -Algorithm sha256).Hash -ne $env:NODE_DOWNLOAD_SHA) { `
        Write-Host 'NODEJS CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    `
    write-output 'downloading vsbuild'; `
    Invoke-WebRequest -UseBasicParsing https://download.visualstudio.microsoft.com/download/pr/12017762/e64d79b40219aea618ce2fe10ebd5f0d/vs_BuildTools.exe -outfile vsbuild.exe; `
    write-output 'installing vsbuild'; `
    Start-Process -FilePath C:\vsbuild.exe -ArgumentList '--quiet','--norestart','--add Microsoft.VisualStudio.Workload.WebBuildTools','--includeRecommended','--includeOptional' -Wait; `
    write-output 'removing vsbuild installer'; `
    Remove-Item -Force vsbuild.exe; `
    write-output 'installing node'; `
    Expand-Archive node.zip -DestinationPath nodejs-tmp; `
    Move-Item nodejs-tmp/node-v${env:NODE_VERSION}-win-x64 "${Env:ProgramFiles}/nodejs"; `
    write-output 'removing node installer'; `
    Remove-Item -Force node.zip; `
    Remove-Item -Force nodejs-tmp; `
    write-output 'sdk install complete';

RUN setx /M PATH $($Env:PATH + ';' + $Env:ProgramFiles + '/nodejs')

# Ensure packages required for standalone apps are pre-fetched
RUN mkdir $env:TEMP\warmup; `
    cd $env:TEMP\warmup; `
    dotnet new web --no-restore; `
    dotnet restore -p:RuntimeIdentifier=win7-x64 `
        --source https://api.nuget.org/v3/index.json; `
    cd ..; `
rm -Recurse -Force $env:TEMP\warmup