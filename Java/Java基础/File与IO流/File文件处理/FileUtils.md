```java
package com.hk.utils;

import org.junit.Test;

import javax.lang.model.element.NestingKind;
import java.io.*;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;
import java.util.Map;
import java.util.zip.ZipEntry;
import java.util.zip.ZipException;
//import org.apache.tools.zip.ZipEntry;
//import org.apache.tools.zip.ZipFile;
import java.util.zip.ZipFile;
import java.util.zip.ZipOutputStream;

/**
 * @Author bigbeardhk
 * @Date 2020/11/25 22:04
 * @Email bigbeardhk@163.com
 */
public class FileUtil {

    public static final String FILE_SEPARATOR = System.getProperty("file.separator");
    private static final int  BUFFER_SIZE = 2 * 1024;

    public static List<String> fileByType = new ArrayList<>();
    public static List<String> fileByName = new ArrayList<>();


//    document directory folder


    /**
     * 删除某个目录下所有文件及文件夹
     *
     * @param srcFile 根目录
     * @return boolean 是否删除成功
     */
    private static boolean delFilesAndFolders(File srcFile) {
        File[] listFiles = srcFile.listFiles();
        if (listFiles == null) {
            return true;
        }
        for (int i = 0; i < listFiles.length; i++) {
            if (listFiles[i].isDirectory()) {
                delFilesAndFolders(listFiles[i]);
            }
            try {
                Files.delete(listFiles[i].toPath());
            } catch (IOException e) {
//                log.error("Delete temp directory or file failed." + e.getMessage());
                return false;
            }
        }
        return true;
    }


    /**
     * 删除目标文件或目标文件夹下的所有文件(不包括文件夹)
     *
     * @param destFile 源文件或根目录
     * @return 是否删除成功
     */
    private static boolean delFiles(File destFile) {

        if (!destFile.exists()) {
            return false;
        } else {

            if (destFile.isFile()) {
                return destFile.delete();
            }

            if (destFile.isDirectory()) {

                File[] listFiles = destFile.listFiles();

                for (int i = 0; i < listFiles.length; i++) {
                    if (!delFiles(listFiles[i])) {
                        //log.error("Delete temp directory or file failed." + e.getMessage());
                        return false;
                    }
                }
            }
            return true;
        }
    }


    /**
     * 文件删除通用入口
     *
     * @param srcFile         源文件或文件夹
     * @param isReserveFolder 是否保留文件夹
     * @return
     */
    public static boolean delect(File srcFile, Boolean isReserveFolder) {

        if (!srcFile.exists()) {
            return false;
        } else {

            if (srcFile.isFile()) {
                return delFiles(srcFile);
            }

            if (srcFile.isDirectory()) {
                if (isReserveFolder) {
                    return delFiles(srcFile);
                } else {
                    return delFilesAndFolders(srcFile) && srcFile.delete();
                }
            }

        }

        return true;
    }


    /**
     * 文件删除通用入口
     *
     * @param srcFile 源文件或文件夹
     * @return
     */
    public static boolean delect(File srcFile) {

        if (!srcFile.exists()) {
            return false;
        } else {

            if (srcFile.isFile()) {
                return delFiles(srcFile);
            }

            if (srcFile.isDirectory()) {
                return delFilesAndFolders(srcFile) && srcFile.delete();
            }

        }

        return true;
    }


    /**
     * 复制文件到指定目录下
     * (传统文件流)
     *
     * @param srcFile 源文件
     * @param destDir 指定目录
     */
    private static boolean copyFileByIO(File srcFile, File destDir) {

        FileInputStream fis = null;
        FileOutputStream bos = null;
        String destFilePath = null;
        try {

            destFilePath = destDir + File.separator + srcFile.getName();

            fis = new FileInputStream(srcFile);

            File destFile = new File(destFilePath);

            if (!destFile.getParentFile().exists()) {
                destFile.mkdirs();
            }

            bos = new FileOutputStream(destFile);

            /**
             * 常规的读写复制
             */
            int len;
            byte[] b = new byte[1024];
            while ((len = fis.read(b)) != -1) {
                bos.write(b, 0, len);
                bos.flush();
            }

        } catch (Exception e) {
            e.printStackTrace();
            return false;
        } finally {
            closeIO(fis, bos);
        }

        return true;

    }

    /**
     * 复制文件到指定目录下--推荐
     * (import java.nio.file)
     *
     * @param srcFile 源文件
     * @param destDir 指定目录
     */
    private static boolean copyFileByNIO(File srcFile, File destDir) {

        if (srcFile.isFile()) {
            //目标文件路劲
            Path destDirPath = Paths.get(destDir.getPath() + File.separator + srcFile.getName());

            if (!destDir.exists()) {
                destDir.mkdirs();
            }

            try {
                Files.copy(srcFile.toPath(), destDirPath);
                return true;
            } catch (IOException e) {
                e.printStackTrace();
                return false;
            }
        }
        return false;
    }


    /**
     * 单个文件的复制到指定目录
     *
     * @param srcFile    源文件
     * @param destDir    指定目录
     * @param copyMethod 单个文件的复制方式
     * @return
     */
    private static boolean copyFile(File srcFile, File destDir, int copyMethod) {

        switch (copyMethod) {
            case 1:
                return copyFileByIO(srcFile, destDir);
            case 2:
                return copyFileByNIO(srcFile, destDir);
            default:
                return copyFileByNIO(srcFile, destDir);
        }

    }


    /**
     * 复制文件或文件夹到指定目录下
     *
     * @param srcFiles 源文件
     * @param destDir  目标文件夹
     */
    private static boolean copyFiles(File srcFiles, File destDir) {

        if (destDir.isFile()) {
            return false;
        } else {

//            boolean isCopyEmptyDir=true;

            if (!destDir.exists()) {
                destDir.mkdirs();
            }

            if (srcFiles.isFile()) {
                return copyFile(srcFiles, destDir, 0);
            }

            if (srcFiles.isDirectory()) {

                File[] listFiles = srcFiles.listFiles();
                String destDirPath = destDir.getPath() + File.separator + srcFiles.getName();

                //空文件夹
                if (listFiles != null && listFiles.length == 0) {

//                    if (!isCopyEmptyDir) {
//                        return false;
//                    }

                    File file = new File(destDirPath);
                    if (!file.exists()) {
                        file.mkdirs();
                    }
                }

                for (int i = 0; i < listFiles.length; i++) {
//                    copyFiles(listFiles[i], new File(destDirPath), isCopyEmptyDir);
                    copyFiles(listFiles[i], new File(destDirPath));
                }

            }
            return true;
        }

    }


    /**
     * 复制文件或文件夹到指定目录下
     *
     * @param srcFile 源文件
     * @param destDir 指定目录
     */
    public static boolean copy(File srcFile, File destDir) {

        if (!srcFile.exists()) {
            return false;
        } else {
            if (srcFile.isFile()) {
                return copyFile(srcFile, destDir, 0);
            }

            if (srcFile.isDirectory()) {
                return copyFiles(srcFile, destDir);
            }
        }

        return false;
    }


    /**
     * 剪切文件指定目录下
     *
     * @param srcFile 源文件
     * @param destDir 指定目录
     */
    public static boolean fileCut(File srcFile, File destDir) {

        if (!srcFile.exists()) {
            return false;
        } else {
            if (srcFile.isFile()) {
                File destFile = new File(destDir.getPath() + File.separator + srcFile.getName());
                return srcFile.renameTo(destFile);
            }
        }
        return false;
    }


    /**
     * 剪切文件或文件夹到指定目录下
     *
     * @param srcFile 源文件
     * @param destDir 指定目录
     */
    public static boolean cut(File srcFile, File destDir) {
        if (!srcFile.exists()) {
            return false;
        } else {
            if (copy(srcFile, destDir)) {
                return delect(srcFile);
            } else {
                delect(destDir);
                return false;
            }
        }
    }


    //===================================第二部分==========================================

    /**
     * 文件或文件夹重命名
     * (不改变文件类型)
     *
     * @param srcFile 目标文件
     * @param rename  文件或文件夹重命名
     * @return
     */
    public static boolean rename(File srcFile, String rename) {
        if (!srcFile.exists()) {
            return false;
        } else {
            String fileName = srcFile.getName();
            int indexOf = fileName.lastIndexOf(".");
            String fileType = "";
            if (indexOf != -1) {
                fileType = fileName.substring(indexOf);
            }
            File renameFile = new File(srcFile.getParent() + File.separator + rename + fileType);
            return srcFile.renameTo(renameFile);
        }
    }


    /**
     * 文件或文件夹重命名
     *
     * @param srcFile 目标文件
     * @param rename  文件或文件夹重命名
     * @return
     */
    public static boolean renameTo(File srcFile, String rename) {
        if (!srcFile.exists()) {
            return false;
        } else {
            File renameFile = new File(srcFile.getParent() + File.separator + rename);
            return srcFile.renameTo(renameFile);
        }
    }


    /**
     * 获取文件后缀名
     * 如,  .doc
     *
     * @param file
     * @return
     */
    public static String getSuffix(File file) {

        if (!file.exists()) {
            //log.info("该文件不存在!")
            return null;
        } else {
            if (file.isFile()) {
                String fileName = file.getName();
                int indexOf = fileName.lastIndexOf(".");
                if (indexOf == -1) {
                    //log.info("该文件没有后缀名!")
                    return "";
                } else {
                    String getSuffix = fileName.substring(indexOf);
                    return getSuffix;
                }
            }
        }
        return null;
    }


    /**
     * 获取文件类型
     *
     * @param file
     * @return
     */
    public static String getFileType(File file) {

        if (!file.exists()) {
            //log.info("该文件不存在!")
            return null;
        } else {
            if (file.isFile()) {
                String fileName = file.getName();
                int indexOf = fileName.lastIndexOf(".");
                if (indexOf == -1) {
                    //log.info("该文件没有后缀名!")
                    return "";
                } else {
                    String getFileType = fileName.substring(indexOf + 1);
                    return getFileType;
                }
            }
        }
        return null;
    }


    /**
     * 获取文件前缀名
     *
     * @param file
     * @return
     */
    public static String getPrefix(File file) {

        if (!file.exists()) {
            //log.info("该文件不存在!")
            return null;
        } else {
            if (file.isFile()) {
                String fileName = file.getName();
                int indexOf = fileName.lastIndexOf(".");
                if (indexOf == -1) {
                    //log.info("该文件没有后缀名!")
                    return fileName;
                } else {
                    String getPrefix = fileName.substring(0, indexOf);
                    return getPrefix;
                }
            }
        }
        return null;
    }

//===================================第三部分==============================================


    /**
     * 查找该目录下的符合类型的文件(只查找这一层的目录)
     *
     * @param file
     * @param fileType
     * @return
     */
    public static List<String> findFileByType(File file, String fileType) {

        if (!file.exists()) {
            return null;
        } else {
            if (file.isDirectory()) {

                List<String> fileByType = new ArrayList<>();

                //【步骤】获取该目录下的所有符合文件
                File[] listFiles = file.listFiles((dir, name) -> {
                    return name.matches(".+\\." + fileType);
                });

                for (File listFile : listFiles) {
                    fileByType.add(listFile.getPath());
                }

                return fileByType;
            }

        }

        return null;
    }


    /**
     * 查找该目录下所有符合文件类型的文件
     *
     * @param file     需要查找的目录
     * @param fileType 需要查找的文件类型 (doc ppt pdf)
     * @return 返回所有符合文件的路径集合
     */
    private static List<String> findFilesByTypeOfReal(File file, String fileType) {

        String regex = ".+\\." + fileType;

        if (!file.exists()) {
            return null;
        } else {

            if (file.isFile() && file.getName().matches(regex)) {
                fileByType.add(file.getPath());
            }

            if (file.isDirectory()) {

                File[] listFiles = file.listFiles();

                for (File listFile : listFiles) {
                    findFilesByTypeOfReal(listFile, fileType);
                }
            }

        }

        return fileByType;
    }


    /**
     * 查找该目录下所有符合文件类型的文件
     *
     * @param file     需要查找的目录
     * @param fileType 需要查找的文件类型 (doc ppt pdf)
     * @return 返回所有符合文件的路径集合
     */
    public static List<String> findFilesByType(File file, String fileType) {

        if(fileByType.size()!=0){
            fileByType.clear();
        }

       return findFilesByTypeOfReal(file,fileType);
    }

    /**
     * 查找该目录下所有符合文件名称的文件
     *
     * @param file     需要查找的目录
     * @param fileName 需要查找的文件名称(test.doc)
     * @return 返回所有符合文件的路径集合
     */
    private static List<String> findFilesByNameOfReal(File file, String fileName) {

        String regex = fileName;

        if (!file.exists()) {
            return null;
        } else {

            if (file.isFile() && file.getName().matches(regex)) {
                fileByName.add(file.getPath());
            }

            if (file.isDirectory()) {

                File[] listFiles = file.listFiles();

                for (File listFile : listFiles) {
                    findFilesByNameOfReal(listFile, fileName);
                }
            }

        }

        return fileByName;
    }


    /**
     * 查找该目录下所有符合文件类型的文件
     *
     * @param file     需要查找的目录
     * @param fileType 需要查找的文件类型 (doc ppt pdf)
     * @return 返回所有符合文件的路径集合
     */
    public static List<String> findFilesByName(File file, String fileType) {

        if(fileByName.size()!=0){
            fileByName.clear();
        }

        return findFilesByNameOfReal(file,fileType);
    }


//========================第四部分  文件的压缩与解压======================================

    /**
     * @param map key:目录名称(下存value文件) value:key下的文件集合
     * @param out  打包后导向的输出流
     * @return  压缩了多少个文件
     */
    public static int toZip(Map<String, List<File>> map, OutputStream out) {
        long start = System.currentTimeMillis();
        ZipOutputStream zos = null;
        FileInputStream in = null;
        int fileNum = 0;
        byte[] buffer = new byte[1024];
        try {
            zos = new ZipOutputStream(out);
            for (String key : map.keySet()) {
                List<File> lists = map.get(key);
                for (int i = 0; i < lists.size(); i++) {
                    if (lists.get(i).exists()) {
                        int len;

                        in = new FileInputStream(lists.get(i));
                        zos.putNextEntry(new ZipEntry(key + File.separator + lists.get(i).getName()));
                        while ((len = in.read(buffer)) != -1) {
                            zos.write(buffer, 0, len);
                        }
                        zos.closeEntry();
                    }
                }
                fileNum += lists.size();
            }
            long end = System.currentTimeMillis();
//            logger.info("压缩完成，耗时：" + (end - start) +" ms");
            System.out.println("压缩完成，耗时：" + (end - start) + " ms");
            return fileNum;
        } catch (IOException e) {
//            logger.error(e.getMessage());
            e.printStackTrace();
        }  finally {
            closeIO(in,zos);
        }
        return -1;
    }


    /**
     * 将srcDir目录下所有文件和文件夹打包为流,导向out输出流中
     * @param srcDir  待压缩的目录
     * @param out     压缩后向外输出的流(一般为压缩的文件流)
     * @throws RuntimeException
     */
    public static void toZip(String srcDir, OutputStream out) throws RuntimeException{
//        boolean KeepDirStructure=true;
        long start = System.currentTimeMillis();
        ZipOutputStream zos = null ;
        try {
            zos = new ZipOutputStream(out);
            File sourceFile = new File(srcDir);
            compress(sourceFile,zos,sourceFile.getName(),true);
            long end = System.currentTimeMillis();
            System.out.println("压缩完成，耗时：" + (end - start) +" ms");
        } catch (Exception e) {
            throw new RuntimeException("zip error from ZipUtils",e);
        }finally{
            if(zos != null){
                try {
                    zos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * toZip的测试用例:
     *   File tozip=new File("C:\\Users\\user\\Desktop\\dfs_test\\upload\\test777\\test.zip");
     *   FileOutputStream fileOutputStream=null;
     *         try {
     *         if(!tozip.exists()){
     *             tozip.createNewFile();
     *         }
     *             fileOutputStream=new FileOutputStream(tozip);
     *             toZip(file7.getPath(),fileOutputStream);
     *         } catch (IOException e) {
     *             e.printStackTrace();
     *         }finally {
     *             if(fileOutputStream!=null){
     *                 try {
     *                     fileOutputStream.close();
     *                 } catch (IOException e) {
     *                     e.printStackTrace();
     *                 }
     *             }
     *         }
     */


    /**
     * 递归压缩方法
     * @param sourceFile       源文件
     * @param zos              zip输出流
     * @param name             压缩后的名称
     * @param KeepDirStructure 是否保留原来的目录结构,true:保留目录结构;
     *                         false:所有文件跑到压缩包根目录下(注意：不保留目录结构可能会出现同名文件,会压缩失败)
     * @throws Exception
     */
    private static void compress(File sourceFile, ZipOutputStream zos, String name, boolean KeepDirStructure) throws Exception {
        byte[] buf = new byte[BUFFER_SIZE];
        if (sourceFile.isFile()) {
            // 向zip输出流中添加一个zip实体，构造器中name为zip实体的文件的名字
            zos.putNextEntry(new ZipEntry(name));
            // copy文件到zip输出流中
            int len;
            FileInputStream in = new FileInputStream(sourceFile);
            while ((len = in.read(buf)) != -1) {
                zos.write(buf, 0, len);
            }
            // Complete the entry
            zos.closeEntry();
            in.close();
        } else {
            File[] listFiles = sourceFile.listFiles();
            if (listFiles == null || listFiles.length == 0) {
                // 需要保留原来的文件结构时,需要对空文件夹进行处理
                if (KeepDirStructure) {
                    // 空文件夹的处理
                    zos.putNextEntry(new ZipEntry(name + "/"));
                    // 没有文件，不需要文件的copy
                    zos.closeEntry();
                }
            } else {
                for (File file : listFiles) {
                    // 判断是否需要保留原来的文件结构
                    if (KeepDirStructure) {
                        // 注意：file.getName()前面需要带上父文件夹的名字加一斜杠,
                        // 不然最后压缩包中就不能保留原来的文件结构,即：所有文件都跑到压缩包根目录下了
                        compress(file, zos, name + "/" + file.getName(), KeepDirStructure);
                    } else {
                        compress(file, zos, file.getName(), KeepDirStructure);
                    }
                }
            }
        }
    }


    /**
     * 解压文件操作
     * @param zipFilePath zip文件路径
     * @param descDir 解压出来的文件保存的目录
     */
    public static void unZip(String zipFilePath,String descDir){
        File zipFile=new File(zipFilePath);
        File pathFile=new File(descDir);

        if(!pathFile.exists()){
            pathFile.mkdirs();
        }
        ZipFile zip=null;
        InputStream in=null;
        OutputStream out=null;

        try {
            zip=new ZipFile(zipFile);
            Enumeration<?> entries=zip.entries();
            while(entries.hasMoreElements()){
                ZipEntry entry=(ZipEntry) entries.nextElement();
                String zipEntryName=entry.getName();
                in=zip.getInputStream(entry);

                String outPath=(descDir+"/"+zipEntryName).replace("\\*", "/");
                //判断路径是否存在，不存在则创建文件路径
                File file=new File(outPath.substring(0, outPath.lastIndexOf('/')));
                if(!file.exists()){
                    file.mkdirs();
                }
                //判断文件全路径是否为文件夹,如果是上面已经创建,不需要解压
                if(new File(outPath).isDirectory()){
                    continue;
                }
                out=new FileOutputStream(outPath);

                byte[] buf=new byte[4*1024];
                int len;
                while((len=in.read(buf))>=0){
                    out.write(buf, 0, len);
                }
                in.close();

                System.out.println("==================解压完毕==================");
            }
        } catch (Exception e) {
            System.out.println("==================解压失败==================");
            e.printStackTrace();
        }finally{
            try {
                if(zip!=null){
                    zip.close();
                }
                if(in!=null){
                    in.close();
                }
                if(out!=null){
                    out.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }


//========================最后部分===========================================

    /**
     * @param fin  输入流
     * @param fout 文件输出流
     */
    public static void closeIO(InputStream fin, OutputStream fout) {
        if (fin != null) {
            try {
                fin.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        if (fout != null) {
            try {
                fout.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }


    public static void main(String[] arg) {

        File file1 = new File("C:\\Users\\user\\Desktop\\dfs_test\\upload\\测试文件夹\\2020095.doc");

        File file2 = new File("C:\\Users\\user\\Desktop\\dfs_test\\upload\\测试文件夹\\fdgf.ppt");

        File file3 = new File("C:\\Users\\user\\Desktop\\dfs_test\\upload\\测试文件夹\\test");


        File file5 = new File("C:\\Users\\user\\Desktop\\dfs_test\\upload\\测试文件夹");

        File file4 = new File("C:\\Users\\user\\Desktop\\dfs_test\\upload\\test777");
        File file8 = new File("C:\\Users\\user\\Desktop\\dfs_test\\upload\\test777\\test.zip");

        File file6 = new File("C:\\Users\\user\\Desktop\\dfs_test\\upload\\test2");

        File file7 = new File("C:\\Users\\user\\Desktop\\dfs_test\\upload\\test5\\测试文件夹");

//        System.out.println(CopyFile(file3, file1));f
//
//        System.out.println(copy(file7, file5));
//        System.out.println(cut(file1, file6));
//        System.out.println(getSuffix(file7));
//        System.out.println(getPrefix(file7));
//        fileCut(file7, file4);
//         cut(file7,file4);//42
//        System.out.println(delFiles(file7));
//        System.out.println(copy(file7, file6));


//        System.out.println(findFilesByType(file7, "doc"));
//        System.out.println(findFilesByName(file7, "1.ppt"));
        File tozip=new File("C:\\Users\\user\\Desktop\\dfs_test\\upload\\test777\\test888.zip");
//                   FileOutputStream fileOutputStream=null;
//            try {
//              if(!tozip.exists()){
//                  tozip.createNewFile();
//              }
//                 fileOutputStream=new FileOutputStream(tozip);
//                  toZip(file7.getPath(),fileOutputStream);
//              } catch (IOException e) {
//                  e.printStackTrace();
//              }finally {
//                 if(fileOutputStream!=null){
//                      try {
//                          fileOutputStream.close();
//                      } catch (IOException e) {
//                         e.printStackTrace();
//     }
//                  }
//              }
        unZip(tozip.getPath(),"C:\\Users\\user\\Desktop\\dfs_test\\upload\\test");


//        System.out.println(copyFiles(file7, file6));
//        System.out.println(delect(file7, false));
//        copyFileByNio(file1,file4);

//        deleteAllFile("C:\\Users\\user\\Desktop\\dfs_test\\upload\\test5\\测试文件夹",true);

//        deleteAllByPath(file7);
//        try {
//            Files.copy(file5.toPath(),file4.toPath());
//        } catch (IOException e) {
//            e.printStackTrace();
//        }


    }

}

```
