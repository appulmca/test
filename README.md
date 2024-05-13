import java.time.LocalDate;
import java.time.YearMonth;
import java.util.List;

public class AddressValidator {
    
    public static boolean validateResidentialHistory(List<Address> residentialHistory) {
        if (residentialHistory.isEmpty()) {
            return false; // Empty residential history
        }

        // Get the current year and month
        YearMonth currentYearMonth = YearMonth.now();

        // Sort the residential history by fromMonthYear
        residentialHistory.sort((a1, a2) -> a1.getFromMonthYear().compareTo(a2.getFromMonthYear()));

        // Check the first address to ensure it doesn't start more than 5 years ago
        if (residentialHistory.get(0).getFromMonthYear().isBefore(currentYearMonth.minusYears(5))) {
            return false; // Address history starts more than 5 years ago
        }

        for (int i = 1; i < residentialHistory.size(); i++) {
            Address previousAddress = residentialHistory.get(i - 1);
            Address currentAddress = residentialHistory.get(i);

            // Check for gap between addresses
            if (previousAddress.getToMonthYear() != null) {
                YearMonth nextExpectedMonth = previousAddress.getToMonthYear().plusMonths(1);
                if (!nextExpectedMonth.equals(currentAddress.getFromMonthYear())) {
                    return false; // Gap between addresses
                }
            }

            // Check for current address
            if (currentAddress.isCurrent()) {
                if (currentAddress.getFromMonthYear().isAfter(currentYearMonth)) {
                    return false; // Current address starts in future
                }
                return true; // Current address found, no need to check further
            }
        }

        // Check the last address to ensure it doesn't extend beyond the current date
        Address lastAddress = residentialHistory.get(residentialHistory.size() - 1);
        if (lastAddress.getToMonthYear() == null && !lastAddress.isCurrent()) {
            return false; // Last address doesn't have an end date
        }

        if (lastAddress.getToMonthYear() != null && lastAddress.getToMonthYear().isAfter(currentYearMonth)) {
            return false; // Last address extends beyond current date
        }

        return true; // Residential history is valid
    }

    public static void main(String[] args) {
        // Example usage
        List<Address> residentialHistory = List.of(
            new Address(YearMonth.of(2019, 4), YearMonth.of(2020, 5)),
            new Address(YearMonth.of(2020, 6), YearMonth.of(2021, 7)),
            new Address(YearMonth.of(2021, 8), null, true)
        );

        boolean isValid = validateResidentialHistory(residentialHistory);
        System.out.println("Residential history is valid: " + isValid);
    }
}

class Address {
    private YearMonth fromMonthYear;
    private YearMonth toMonthYear;
    private boolean current;

    public Address(YearMonth fromMonthYear, YearMonth toMonthYear) {
        this.fromMonthYear = fromMonthYear;
        this.toMonthYear = toMonthYear;
        this.current = false;
    }

    public Address(YearMonth fromMonthYear, YearMonth toMonthYear, boolean current) {
        this.fromMonthYear = fromMonthYear;
        this.toMonthYear = toMonthYear;
        this.current = current;
    }

    public YearMonth getFromMonthYear() {
        return fromMonthYear;
    }

    public YearMonth getToMonthYear() {
        return toMonthYear;
    }

    public boolean isCurrent() {
        return current;
    }
}
