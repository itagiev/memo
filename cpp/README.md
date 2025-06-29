### Настройки

**WARNING** Set Configuration: "All Configurations" and Platforms: "All Platforms"

General
bin directory: $(SolutionDir)bin\$(Platform)\$(Configuration)\
out directory: $(SolutionDir)out\$(Platform)\$(Configuration)\

C/C++ -> General
Warning level: Level4 (/W4)
Treat warnings as errors: Yes/No

C/C++ -> Language
Conformance mode: Yes (/permissive-)

C/C++ -> Language
C++ Language Standard: 14/17/20/latest

### Static linking example

C/C++ -> General
Additional include directories: include header files (e.g. $(SolutionDir)dependencies\include)

Linker -> General
Additional library directories: include .lib files (e.g. $(SolutionDir)\dependencies\lib-vc2022)

Linker -> Input
Additional dependencies: (e.g. User32.lib;opengl32.lib;glfw3.lib...)

P.S.
C/C++ -> Code Generation -> Runtime Libarary (Здесь можно настроить параметры /MD, /MT, /MDd, /MTd)
