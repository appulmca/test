import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.Query;
import javax.transaction.Transactional;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/execute-query")
public class DatabaseQueryController {

    @PersistenceContext
    private EntityManager entityManager;

    @PostMapping
    @Transactional
    public ResponseEntity<String> executeQuery(@RequestBody QueryRequest request) {
        // Validate request parameters
        if (!isValidRequest(request)) {
            return ResponseEntity.badRequest().body("Invalid request parameters");
        }

        // Construct and execute the database query
        String query = constructQuery(request);
        try {
            Query nativeQuery = entityManager.createNativeQuery(query);
            // Bind parameters
            bindParameters(nativeQuery, request.getWhereParams());
            int result = nativeQuery.executeUpdate();
            return ResponseEntity.ok("Query executed successfully. Affected rows: " + result);
        } catch (Exception e) {
            // Handle exception
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Error executing database query");
        }
    }

    private boolean isValidRequest(QueryRequest request) {
        // Validate request parameters (add appropriate validation logic)
        return request != null && request.getOperation() != null && request.getTableName() != null
                && !request.getWhereParams().isEmpty();
    }

    private String constructQuery(QueryRequest request) {
        // Construct the database query using native SQL
        StringBuilder queryBuilder = new StringBuilder();
        queryBuilder.append(request.getOperation()).append(" FROM ").append(request.getTableName()).append(" WHERE ");

        // Add WHERE conditions for each parameter
        for (int i = 0; i < request.getWhereParams().size(); i++) {
            WhereClauseParam param = request.getWhereParams().get(i);
            if (i > 0) {
                queryBuilder.append(" AND ");
            }
            queryBuilder.append(param.getName()).append(" = ?");
        }

        return queryBuilder.toString();
    }

    private void bindParameters(Query query, List<WhereClauseParam> params) {
        // Bind parameters to the query
        for (int i = 0; i < params.size(); i++) {
            WhereClauseParam param = params.get(i);
            query.setParameter(i + 1, param.getValue());
        }
    }
}
