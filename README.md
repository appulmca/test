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

        // Handle overlapping and fill gaps programmatically
        List<Employment> processedEmployments = new ArrayList<>();
        YearMonth lastEndDate = tenYearsAgo;
        for (Employment employment : filteredEmployments) {
            if (lastEndDate.isBefore(employment.getFromDate().minusMonths(1))) {
                processedEmployments.add(new Employment(lastEndDate.plusMonths(1), employment.getFromDate().minusMonths(1), Action.ADDED));
            } else if (lastEndDate.isAfter(employment.getFromDate())) {
                employment.setFromDate(lastEndDate.plusMonths(1));
            }
            processedEmployments.add(employment);
            lastEndDate = employment.getToDate() == null ? YearMonth.from(now) : employment.getToDate();
        }

        // Check for the final gap
        if (lastEndDate.isBefore(YearMonth.from(now))) {
            processedEmployments.add(new Employment(lastEndDate.plusMonths(1), YearMonth.from(now), Action.ADDED));
        }

        // Check for the initial gap
        if (processedEmployments.isEmpty() || processedEmployments.get(0).getFromDate().isAfter(tenYearsAgo)) {
            YearMonth fillToDate = processedEmployments.isEmpty() ? YearMonth.from(now) : processedEmployments.get(0).getFromDate().minusMonths(1);
            processedEmployments.add(0, new Employment(tenYearsAgo, fillToDate, Action.ADDED));
        }

        // Sort again to maintain order
        Collections.sort(processedEmployments, Comparator.comparing(Employment::getFromDate));

        return new EmploymentResponse(processedEmployments, null);
    }
