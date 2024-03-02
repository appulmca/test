import java.util.List;

class QueryRequest {
    private String operation;
    private String tableName;
    private List<WhereClauseParam> whereParams;


}

class WhereClauseParam {
    private String name;
    private String value;
    private String type; // Add type field

}
