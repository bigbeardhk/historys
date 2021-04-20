```java
public static void downloadFile(File file, HttpServletResponse response, boolean isDelete,String fileName) throws IOException {
        BufferedInputStream fis = null;
        OutputStream toClient = null;
        try {
            // 以流的形式下载文件。
//            fileName = fileName+".zip";
            fis = new BufferedInputStream(new FileInputStream(file.getPath()));
            toClient = new BufferedOutputStream(response.getOutputStream());
            byte[] buffer = new byte[fis.available()];
            fis.read(buffer);
            // 清空response
            response.reset();
//            response.setContentType("application/octet-stream");
            response.setHeader("Access-Control-Allow-Origin", "*");
            response.setHeader("Access-Control-Expose-Headers", "Content-Disposition");
            response.setContentType("application/json;charset=UTF-8");
            response.setHeader("Content-disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8"));
            toClient.write(buffer);
            toClient.flush();
            toClient.close();
            response.setStatus(200);
            response.flushBuffer();
        }
        catch (IOException ex) {
            logger.error(ex.getMessage());
            // 抛异常，用于返回调用结果
            throw new IOException();
        }finally {
            if(toClient != null) {
                try {
                    toClient.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            //是否将生成的服务器端文件删除
            if(isDelete)
            {
                file.delete();
            }
        }
    }
```
