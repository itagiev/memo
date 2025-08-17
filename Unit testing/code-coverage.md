## Code coverage

### Setup

Установить расширения для visual studio. Далее перейти в опции найти Run Coverlet Report и в Options, в Integration type выбрать Collector

Перейти в корневую папку проекта. У выполнить следующие команды:
- dotnet tool install -g dotnet-reportgenerator-globaltool
- dotnet tool install dotnet-reportgenerator-globaltool --tool-path tools
- dotnet new tool-manifest
- dotnet tool install dotnet-reportgenerator-globaltool

### Использование

В tools вызываем Run Code Coverage. Также можно включить Code Coverage Highlighting