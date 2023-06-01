to push telemetry from the docker container running localy to App Insights, make the following changes in docker-compose and re-build the application:

rockpaperscissors-server:
    build:
      context: .
      dockerfile: Dockerfile-Server
    container_name: rockpaperscissors-server
    environment:
      "ConnectionStrings:DefaultConnection": "Server=rockpaperscissors-sql,1433;Database=RockPaperScissorsBoom;User Id=SA;Password=<Password Here>;"
      ASPNETCORE_HOSTINGSTARTUPASSEMBLIES: "Microsoft.AspNetCore.ApplicationInsights.HostingStartup"
      "ApplicationInsights:InstrumentationKey": "<AppInsights Instrumentation Key here-delete the parenthesis>"
