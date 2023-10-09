import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class EmployeeWorkAnalyzer {

    // Function to parse the date-time string and return a Date object
    public static Date parseDatetime(String datetimeStr) throws ParseException {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        return dateFormat.parse(datetimeStr);
    }

    // Function to calculate the time difference between two Date objects in hours
    public static double calculateHoursDifference(Date startTime, Date endTime) {
        long timeDifference = endTime.getTime() - startTime.getTime();
        return timeDifference / (60.0 * 60.0 * 1000.0);
    }

    // Function to check if an employee has worked for 7 consecutive days
    public static boolean hasConsecutiveDays(List<Map<String, Object>> workSchedule) {
        int consecutiveDaysCount = 0;
        for (int i = 0; i < workSchedule.size() - 1; i++) {
            Date currentDate = (Date) workSchedule.get(i).get("datetime");
            Date nextDate = (Date) workSchedule.get(i + 1).get("datetime");
            long dayDifference = (nextDate.getTime() - currentDate.getTime()) / (24 * 60 * 60 * 1000);
            if (dayDifference == 1) {
                consecutiveDaysCount++;
            } else {
                consecutiveDaysCount = 0;
            }

            if (consecutiveDaysCount >= 6) {
                return true;
            }
        }
        return false;
    }

    // Function to analyze the work schedule and print the required information
    public static void analyzeWorkSchedule(String filePath) throws IOException, ParseException {
        BufferedReader reader = new BufferedReader(new FileReader(filePath));
        String line;
        Map<String, List<Map<String, Object>>> employees = new HashMap<>();

        while ((line = reader.readLine()) != null) {
            String[] data = line.split(",");
            String employeeName = data[0];
            Date datetime = parseDatetime(data[1]);
            double hoursWorked = Double.parseDouble(data[2]);

            if (!employees.containsKey(employeeName)) {
                employees.put(employeeName, new ArrayList<>());
            }

            Map<String, Object> entry = new HashMap<>();
            entry.put("datetime", datetime);
            entry.put("hoursWorked", hoursWorked);
            employees.get(employeeName).add(entry);
        }
        reader.close();

        for (Map.Entry<String, List<Map<String, Object>>> entry : employees.entrySet()) {
            String employeeName = entry.getKey();
            List<Map<String, Object>> workSchedule = entry.getValue();
            boolean consecutiveDays = hasConsecutiveDays(workSchedule);
            for (int i = 0; i < workSchedule.size() - 1; i++) {
                Date startTime = (Date) workSchedule.get(i).get("datetime");
                Date endTime = (Date) workSchedule.get(i + 1).get("datetime");
                double hoursDifference = calculateHoursDifference(startTime, endTime);
                if (consecutiveDays) {
                    System.out.println(employeeName + " worked for 7 consecutive days.");
                    break;
                }
                if (1 < hoursDifference && hoursDifference < 10) {
                    System.out.println(employeeName + " has less than 10 hours of time between shifts but greater than 1 hour.");
                }
                if ((double) workSchedule.get(i).get("hoursWorked") > 14) {
                    System.out.println(employeeName + " worked for more than 14 hours in a single shift.");
                }
            }
        }
    }

    public static void main(String[] args) throws IOException, ParseException {
        String filePath = "employee_schedule.txt"; // Replace with the path to your input file
        analyzeWorkSchedule(filePath);
    }
}
