import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.Date;

public class YourClass {

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss z")
    private Date dateTime;

    // getters and setters

    public Date getDateTime() {
        return dateTime;
    }

    public void setDateTime(Date dateTime) {
        this.dateTime = dateTime;
    }
}

public class Example {
    public static void main(String[] args) throws Exception {
        String jsonResponse = "{\"dateTime\": \"2024-05-14 15:30:45 EST\"}";

        ObjectMapper objectMapper = new ObjectMapper();
        YourClass yourClass = objectMapper.readValue(jsonResponse, YourClass.class);

        // Print the parsed date and its class type
        System.out.println("Parsed Date: " + yourClass.getDateTime());
        System.out.println("Date Class Type: " + yourClass.getDateTime().getClass().getName());
    }
}
