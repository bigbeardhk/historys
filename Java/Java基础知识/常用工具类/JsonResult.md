简介:
> - 后端统一返回前端的对象封装<p>
> - 当然也可以做为方法的返回体,封装数据与布尔逻辑
  
  
```java

@Data
@ApiModel(value = "JsonResult" , description = "系统接口统一返回实体")
public class JsonResult<T> implements Serializable {

    @ApiModelProperty(value = "返回状态 200 接口返回正常")
    private Integer resultStatus;

    @ApiModelProperty(value = "返回提示")
    private String resultMsg;

    @ApiModelProperty(value = "返回实体")
    private T resultData;

    @ApiModelProperty(value = "页码")
    private Integer page;

    @ApiModelProperty(value = "单页行数")
    private Integer size;

    @ApiModelProperty(value = "总数据量")
    private Long dataTotal;

    @ApiModelProperty(value = "总页数")
    private Integer pageTotal;

    @ApiModelProperty(value = "分页实体")
    private PageVO pageVO;

    /**
     * 单例化
     */
    private JsonResult(){
    }

    /**
     * 成功返回值，只包含正常返回编码，不包含提示信息
     * @return 返回实体
     */
    public static <T> JsonResult<T> successReturn(){
        JsonResult JsonResult = new JsonResult();
        JsonResult.setResultStatus(HttpStatus.OK.value());
        return JsonResult;
    }

    /**
     * 成功返回值，只包含正常返回编码，包含提示信息
     * @param resultMsg 返回参数
     * @return
     */
    public static <T> JsonResult<T> successReturnByMsg(String resultMsg){
        JsonResult JsonResult = new JsonResult();
        JsonResult.setResultStatus(HttpStatus.OK.value());
        JsonResult.setResultMsg(resultMsg);
        return JsonResult;
    }

    /**
     * 成功返回值，只包含正常返回编码，包含提示信息
     * @param <T>
     * @return
     */
    public static <T> JsonResult<T> successReturnByData(String resultMsg , T data){
        JsonResult JsonResult = new JsonResult();
        JsonResult.setResultStatus(HttpStatus.OK.value());
        JsonResult.setResultMsg(resultMsg);
        JsonResult.setResultData(data);
        return JsonResult;
    }

    /**
     * 成功返回值，只包含正常返回编码，包含提示信息
     * @param <T>
     * @return
     */
    public static <T> JsonResult<T> successReturnByPage(PagedResult<?> pageInfo){
        JsonResult JsonResult = new JsonResult();
        JsonResult.setResultStatus(HttpStatus.OK.value());

        JsonResult.setResultData(pageInfo.getResult());
        // 填充page对象中对应的各个分页参数
        JsonResult.setPageVO(pageInfo.getPageVO());

        return JsonResult;
    }

    /**
     * 成功返回值，只包含正常返回编码，不包含提示信息
     * @param <T>
     * @return
     */
    public static <T> JsonResult<T> successReturnByData(T data){
        JsonResult JsonResult = new JsonResult();
        JsonResult.setResultStatus(HttpStatus.OK.value());
        JsonResult.setResultData(data);
        return JsonResult;
    }

    /**
     * 接口异常返回值，只包含异常返回编码，包含提示信息
     * @param <T>
     * @return
     */
    public static <T> JsonResult<T> errorReturnByMsg(String resultMsg){
        JsonResult JsonResult = new JsonResult();
        JsonResult.setResultStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());
        JsonResult.setResultMsg(resultMsg);
        return JsonResult;
    }

    /**
     * 接口异常返回值，只包含异常返回编码，包含提示信息
     * @param <T>
     * @return
     */
    public static <T> JsonResult<T> errorReturnByMsg(Integer errorCode , String resultMsg){
        JsonResult JsonResult = new JsonResult();
        JsonResult.setResultStatus(errorCode);
        JsonResult.setResultMsg(resultMsg);
        return JsonResult;
    }

    /**
     * 接口异常返回值，只包含异常返回编码，包含提示信息以及异常实体
     * @param <T>
     * @return
     */
    public static <T> JsonResult<T> errorReturnByData(String resultMsg , T data){
        JsonResult JsonResult = new JsonResult();
        JsonResult.setResultStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());
        JsonResult.setResultMsg(resultMsg);
        JsonResult.setResultData(data);
        return JsonResult;
    }

    /**
     * 接口异常返回值，只包含异常返回编码，包含提示异常实体
     * @param <T>
     * @return
     */
    public static <T> JsonResult<T> errorReturnByData(T data){
        JsonResult JsonResult = new JsonResult();
        JsonResult.setResultStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());
        JsonResult.setResultData(data);
        return JsonResult;
    }

    /**
     * 状态码是否为success
     * @return
     */
    public boolean isSuccess(){
        if(HttpStatus.OK.value() == this.getResultStatus().intValue()){
            return true;
        }
        return false;
    }

}



```
