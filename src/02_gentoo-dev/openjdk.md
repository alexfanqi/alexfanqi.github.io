首先要理解为啥有各种各样的jdk
<https://stackoverflow.com/questions/52431764/difference-between-openjdk-and-adoptium-adoptopenjdk>

截止2021-7-22日，华为给bishengjdk加上了riscv支持，目前还在整合到上游openjdk中。

先试一试native下手动安装，大部分库一般gentoo都有,可以emerge -1a openjdk11看看依赖，或者直接--onlydeps，自动安装依赖。

由于jdk自身有一大部分是java写的，所以需要一个现成的“seed jdk”，这需要先打包一个jdk-bin。所以暂时搁置。有关的东西可以瞧瞧icetea。
