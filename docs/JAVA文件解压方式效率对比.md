#JAVA文件解压大文件效率对比

##本机环境

>  型号名称：	MacBook Pro			
>  型号标识符：	MacBookPro17,1		
>  芯片：	Apple M1	
>  核总数：	8（4性能和4效率）	
>  内存：	16 GB	
>  解压文件大小：  1.31 GB		
> JDK 11


###使用BufferdInputStream

```
private static void unzipByStream(String source,String target) throws IOException {
    ZipFile zf = new ZipFile(source);
    List<String> collectPath = Collections.list(zf.getEntries()).stream().filter(ze -> !ze.isDirectory()&&!ze.getName()
                    .startsWith("__MACOSX")).map(org.apache.tools.zip.ZipEntry::getName)
            .collect(Collectors.toList());
    for (String fileName : collectPath) {
        InputStream inputStream = zf.getInputStream(zf.getEntry(fileName));
        BufferedInputStream bufferedInputStream = new BufferedInputStream(inputStream);
        OutputStream outputStream = new FileOutputStream(target + getFullName(fileName));
    
        byte[] buffer = new byte[1048576];
        int read;
        while ((read = bufferedInputStream.read(buffer, 0, buffer.length)) != -1) {
            outputStream.write(buffer, 0, read);
        }
    }
    zf.close();
}
```
###使用apache.tools.ant工具包
```
public static void unzipByApacheTools(String zipFilepath, String destDir) throws Exception {
    Project proj = new Project();
    Expand expand = new Expand();
    expand.setProject(proj);
    expand.setTaskType("unzip");
    expand.setTaskName("unzip");
    expand.setEncoding("UTF-8");
    expand.setSrc(new File(zipFilepath));
    expand.setDest(new File(destDir));
    expand.execute();
}
```
 耗时对比

```
StopWatch '文件解压耗时': running time = 26057948833 ns
---------------------------------------------
ns         %     Task name
---------------------------------------------
16170785417  062%  apache.tools.ant
9887163416  038%  bufferedStream
```
###结论：


***
共耗时 26s 左右，BufferdInputStream耗时 9s，apache.tools.ant 耗时 13s

Expand 解压方式编码更简单，但是BufferedInputStream解压大文件的效率比使用apache.tools.ant提供的Expand方式效率要快1倍以上。
***