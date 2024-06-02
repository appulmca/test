import java.time.LocalDate;
import java.time.YearMonth;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Collectors;

public class EmploymentService {

    public static class Employment {
        private YearMonth fromDate;
        private YearMonth toDate; // null if it's the current employment
        private Action action;

        // Constructors, getters, and setters
        public Employment(YearMonth fromDate, YearMonth toDate, Action action) {
            this.fromDate = fromDate;
            this.toDate = toDate;
            this.action = action;
        }

        public YearMonth getFromDate() {
            return fromDate;
        }

        public void setFromDate(YearMonth fromDate) {
            this.fromDate = fromDate;
        }

        public YearMonth getToDate() {
            return toDate;
        }

        public void setToDate(YearMonth toDate) {
            this.toDate = toDate;
        }

        public Action getAction() {
            return action;
        }

        public void setAction(Action action) {
            this.action = action;
        }
    }

    public enum Action {
        ADDED, UPDATED, DELETED
    }

    public static class EmploymentResponse {
        private List<Employment> employments;
        private String error;

        // Constructors, getters, and setters
        public EmploymentResponse(List<Employment> employments, String error) {
            this.employments = employments;
            this.error = error;
        }

        public List<Employment> getEmployments() {
            return employments;
        }

        public void setEmployments(List<Employment> employments) {
            this.employments = employments;
        }

        public String getError() {
            return error;
        }

        public void setError(String error) {
            this.error = error;
        }
    }

    public EmploymentResponse processEmploymentHistory(List<Employment> employments) {
        // Filter out deleted employments
        List<Employment> filteredEmployments = employments.stream()
                .filter(e -> e.getAction() != Action.DELETED)
                .collect(Collectors.toList());

        // Sort the employments by fromDate
        Collections.sort(filteredEmployments, Comparator.comparing(Employment::getFromDate));

        // Validate dates
        LocalDate now = LocalDate.now();
        YearMonth tenYearsAgo = YearMonth.from(now).minusYears(10);

        for (Employment employment : filteredEmployments) {
            if (employment.getFromDate() == null) {
                return new EmploymentResponse(employments, "From date is missing for one of the employments.");
            }
            if (employment.getToDate() == null && employment.getFromDate().isBefore(tenYearsAgo)) {
                return new EmploymentResponse(employments, "Current employment cannot start before 10 years ago.");
            }
            if (employment.getToDate() != null && employment.getFromDate().isAfter(employment.getToDate())) {
                return new EmploymentResponse(employments, "From date is after to date for one of the employments.");
            }
        }

        // Fill gaps programmatically
        List<Employment> filledEmployments = new ArrayList<>();
        YearMonth lastEndDate = tenYearsAgo;
        for (Employment employment : filteredEmployments) {
            if (lastEndDate.isBefore(employment.getFromDate().minusMonths(1))) {
                filledEmployments.add(new Employment(lastEndDate.plusMonths(1), employment.getFromDate().minusMonths(1), Action.ADDED));
            }
            filledEmployments.add(employment);
            lastEndDate = employment.getToDate() == null ? YearMonth.from(now) : employment.getToDate();
        }

        // Check for the final gap
        if (lastEndDate.isBefore(YearMonth.from(now))) {
            filledEmployments.add(new Employment(lastEndDate.plusMonths(1), YearMonth.from(now), Action.ADDED));
        }

        // Check for the initial gap
        if (filledEmployments.isEmpty() || filledEmployments.get(0).getFromDate().isAfter(tenYearsAgo)) {
            YearMonth fillToDate = filledEmployments.isEmpty() ? YearMonth.from(now) : filledEmployments.get(0).getFromDate().minusMonths(1);
            filledEmployments.add(0, new Employment(tenYearsAgo, fillToDate, Action.ADDED));
        }

        return new EmploymentResponse(filledEmployments, null);
    }

    public static void main(String[] args) {
        EmploymentService service = new EmploymentService();

        List<Employment> employments = new ArrayList<>();
        employments.add(new Employment(YearMonth.of(2022, 1), YearMonth.of(2022, 12), Action.ADDED));
        employments.add(new Employment(YearMonth.of(2024, 1), null, Action.ADDED));

        EmploymentResponse response = service.processEmploymentHistory(employments);
        response.getEmployments().forEach(emp -> {
            System.out.println("From: " + emp.getFromDate() + ", To: " + emp.getToDate() + ", Action: " + emp.getAction());
        });

        if (response.getError != null) {
            System.out.println("Error: " + response.getError());
        }
    }
}
