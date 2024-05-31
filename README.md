import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class FormRulesService {

    private final FormRulesConfig formRulesConfig;

    @Autowired
    public FormRulesService(FormRulesConfig formRulesConfig) {
        this.formRulesConfig = formRulesConfig;
    }

    public boolean isValidationEnabled(String status, String role) {
        List<FormRulesConfig.RoleRule> rules = formRulesConfig.getValidations().get(status);
        if (rules == null) return false;
        return rules.stream()
                .anyMatch(r -> r.getRole().equals(role) && r.isEnabled());
    }

    public boolean isButtonEnabled(String status, String role) {
        List<FormRulesConfig.RoleRule> rules = formRulesConfig.getButtonEnabled().get(status);
        if (rules == null) return false;
        return rules.stream()
                .anyMatch(r -> r.getRole().equals(role) && r.isEnabled());
    }
}
