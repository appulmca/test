        // Validate that fromDate is not null
        for (Employment employment : filteredEmployments) {
            if (employment.getFromDate() == null) {
                return new EmploymentResponse(employments, "From date is missing for one of the employments.");
            }
        }
