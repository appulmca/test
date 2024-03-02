import java.util.List;

class QueryRequest {
    private String operation;
    private String tableName;
    private List<WhereClauseParam> whereParams;

    // Getters and setters
}

class WhereClauseParam {
    private String name;
    private String value;
    private String type; // Add type field

    // Getters and setters
}
