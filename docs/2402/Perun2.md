<!--yml

category: 未分类

date: 2024-05-27 14:49:40

-->

# Perun2

> 来源：[https://perun2.org/](https://perun2.org/)

C#

foreach

(

var

d

in

Directory.GetDirectories(

"c://some//path"

))

{

if

(!Directory.EnumerateFiles(d,

"*.mp3"

) .Any())

{

DirectoryInfo di = Directory.CreateDirectory(d);

di.Attributes = FileAttributes.Directory | FileAttributes.Hidden;

}

}
