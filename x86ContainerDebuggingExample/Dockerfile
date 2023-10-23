# escape=`

# installer stage based on: https://github.com/dotnet/dotnet-docker/blob/5a56c3fcdd874b23498f5df40df3bd0d2c3d231f/src/runtime/7.0/nanoserver-ltsc2022/amd64/Dockerfile
# hash from: https://dotnetcli.blob.core.windows.net/dotnet/release-metadata/7.0/releases.json

FROM mcr.microsoft.com/windows/servercore:ltsc2022-amd64 AS installer
# Retrieve .NET Runtime
RUN powershell -Command `
        $ErrorActionPreference = 'Stop'; `
        $ProgressPreference = 'SilentlyContinue'; `
        `
        $dotnet_version = '7.0.12'; `
        Invoke-WebRequest -OutFile dotnet.zip https://dotnetcli.azureedge.net/dotnet/Runtime/$dotnet_version/dotnet-runtime-$dotnet_version-win-x86.zip; `
        $dotnet_sha512 = '760f4483b9848c2165be30ad4623de4099529fdcbf61c4e6c1738dfe22e3d776ca7ced89e4d7485dc0337c988b1c3bf1c5757a01e97d156df4ad0cf848226244'; `
        if ((Get-FileHash dotnet.zip -Algorithm sha512).Hash -ne $dotnet_sha512) { `
            Write-Host 'CHECKSUM VERIFICATION FAILED!'; `
            exit 1; `
        }; `
        `
        mkdir dotnet; `
        tar -oxzf dotnet.zip -C dotnet; `
        Remove-Item -Force dotnet.zip

FROM mcr.microsoft.com/dotnet/runtime:7.0.12-windowsservercore-ltsc2022 AS base
# Copy the x86 runtime from the installer stage
COPY --from=installer ["/dotnet", "/Program Files (x86)/dotnet"]
WORKDIR /app

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["x86ContainerDebuggingExample/x86ContainerDebuggingExample.csproj", "x86ContainerDebuggingExample/"]
RUN dotnet restore "x86ContainerDebuggingExample/x86ContainerDebuggingExample.csproj"
COPY . .
WORKDIR "/src/x86ContainerDebuggingExample"
RUN dotnet build "x86ContainerDebuggingExample.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "x86ContainerDebuggingExample.csproj" -c Release -o /app/publish /p:UseAppHost=false /p:SelfContained=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "x86ContainerDebuggingExample.dll"]