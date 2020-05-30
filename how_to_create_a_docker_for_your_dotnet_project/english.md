##Dotnet Api on Docker

.Net core is cross platform, so we can run our webapi inside a linux container.

Here we will not use the self-contained strategy to deploy our app.
Although the name suggests .net webapi depends on some library, see some of then in each Distro 
- **Debian**: [libicu63](https://packages.debian.org/sid/libicu63)
- **Ubuntu**: [icu-devtools](https://packages.ubuntu.com/focal/icu-devtools)
- **Alpine**: [libstdc++](https://pkgs.alpinelinux.org/package/edge/main/x86/libstdc++), [libintl](https://pkgs.alpinelinux.org/package/edge/main/x86/libintl) and [icu](https://pkgs.alpinelinux.org/package/edge/main/x86/icu)


First lets create a dotnet project called MyProject, using a terminal or Powershell
```
dotnet new webapi -o MyProject
cd  MyProject
```

to see your project running you can execute
```
dotnet run
```
and open the browser on http://localhost:5000/WeatherForecast

Now we can finally create our Dockerfile<br />
*yes, dockerfile's do **not** have any extension or dot in the name*
```
//Linux or Mac
touch Dockerfile

//Windows - Powershell
New-Item Dockerfile -ItemType file
```

Inside the dockerfile let's paste this content
```dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:3.1-alpine as build-stage
WORKDIR /app
COPY . .
RUN dotnet publish -o publish -c Release -r linux-musl-x64

FROM alpine
RUN apk update && apk add libstdc++ && apk add libintl && apk add icu
COPY --from=build-stage /app/publish /app

ENV ASPNETCORE_URLS=http://+:80

EXPOSE 80

ENTRYPOINT ["/app/MyProject"]
```

and to create the image and run it, execute:
```
docker build -t my-project-image .
docker run -d --name my-project-container -p 5000:80 my-project-image
```
and we can see the results on the browser http://localhost:5000/WeatherForecast

###Explaining the dockerfile

In the default example of docker website he uses the runtime to run the docker which is spending more than the needed using the self-container strategy.<br />
So the hole point in here is to use dotnet libraries only on the step needed: **build-stage** and to serve our application we only use the linux alpine container

So let's break down our dockerfile:

`FROM mcr.microsoft.com/dotnet/core/sdk:3.1-alpine as build-stage`<br/>
here we define the sdk as source image to build our project and define this step as `build-stage`

`WORKDIR /app`<br/>
change the folder that we will apply the following commands

`COPY . .`<br/>
copy our project files to the `/app` folder inside the container

`RUN dotnet publish -o publish -c Release -r linux-musl-x64`<br/>
here we are building and generating a release version inside the `publish` folder and defining our target architecture to be `linux-musl-x64` which is the one that works with alpine container

`FROM alpine`<br/>
here we are defining our final container to be the alpine linux

`RUN apk update && apk add libstdc++ && apk add libintl && apk add icu`<br/>
as mentioned early although the name of this publishing strategy is `self-contained` the dotnet webapi need this packages to work on an alpine linux distro. So we are installing the dotnet core dependencies.

`COPY --from=build-stage /app/publish /app`<br/>
here we are copying the compiled released version from the previous step to the `/app` folder

`ENV ASPNETCORE_URLS=http://+:80`<br/>
dotnet application reads this environment variable to define in which port he will run, so we are defining that we will be listening in the port 80 for any requests

`EXPOSE 80`<br/>
we are making explicit that we are exposing and can respond on the port 80

`ENTRYPOINT ["/app/MyProject"]`<br/>
and here we are defining where is the executable that will start the application


###Advanced

####Tests
One the things that we can do before publishing our project is to run our tests using the same dotnet version that will be used inside the container. So one of the strategies is to run the tests inside the dockerfile ***not recommended*** for big projects or that have a robust CI/CD strategy.

So let's create a more robust example with a test project
```
cd .. && mkdir MyWebApi && mv ./MyProject/ ./MyWebApi/ && cd MyWebApi && mv MyProject/Dockerfile .
dotnet new xunit -o MyProjectTests
```

So on the class, already created by the xunit template, UnitTest1.cs let's create a example test `verify if 2 times 3 is equals to 6`
```c#
using System;
using Xunit;

namespace MyProjectTests
{
    public class UnitTest1
    {
        [Fact]
        public void Test1()
        {
            Assert.Equal(2*3, 6);
        }
    }
}
```

We can run our test using the command on the terminal or Powershell
```
dotnet test
```


On the Dockerfile we need to change our configuration to be as showed bellow
```dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:3.1-alpine as build-stage
WORKDIR /app
COPY . .
RUN dotnet test
RUN dotnet publish -o publish -c Release -r linux-musl-x64 MyProject

FROM alpine
RUN apk update && apk add libstdc++ && apk add libintl && apk add icu
COPY --from=build-stage /app/publish /app

ENV ASPNETCORE_URLS=http://+:80

EXPOSE 80

ENTRYPOINT ["/app/MyProject"]
```
everything that we need was to run the tests before publishing and defining the path to the publish Project `MyProject` on the publish command

###Conclusion

To better manage disk space we use the self-container strategy and the alpine distro.<br/>
To make sure our environment and build will work we run the tesest inside the docker within the same version that will be used to publish.