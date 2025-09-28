# Attendance-system.java
 This is a console-based Java application to manage student attendance for multiple subjects in a school/college environment. The program allows registering students, marking attendance, and viewing attendance reports in a human-readable format. It supports subject-wise lectures and calculates the overall attendance percentage across all subjects

.
*********Student Attendance System*******
                  Project

import java.util.*;
import java.time.LocalDate;

// Class representing a student
class Student {
    String studentName;
    int rollNumber;

    // Subject-wise records: lectures attended and lectures conducted
    HashMap<String, HashMap<Integer, List<LocalDate>>> lecturesAttended;
    HashMap<String, HashMap<Integer, List<LocalDate>>> lecturesConducted;

    // Constructor initializes student details and subjects
    Student(String studentName, int rollNumber, List<String> subjects) {
        this.studentName = studentName;
        this.rollNumber = rollNumber;
        lecturesAttended = new HashMap<>();
        lecturesConducted = new HashMap<>();
        for (String subject : subjects) {
            lecturesAttended.put(subject, new HashMap<>());
            lecturesConducted.put(subject, new HashMap<>());
        }
    }

    // Marks attendance for a lecture
    void markAttendance(String subject, int lectureNumber, boolean isPresent, LocalDate lectureDate) {
        lecturesConducted.get(subject).putIfAbsent(lectureNumber, new ArrayList<>());
        lecturesConducted.get(subject).get(lectureNumber).add(lectureDate);

        if (isPresent) {
            lecturesAttended.get(subject).putIfAbsent(lectureNumber, new ArrayList<>());
            lecturesAttended.get(subject).get(lectureNumber).add(lectureDate);
        }
    }

    // Returns total lectures attended for a subject
    int totalLecturesAttended(String subject) {
        int total = 0;
        for (List<LocalDate> dates : lecturesAttended.get(subject).values()) {
            total += dates.size();
        }
        return total;
    }

    // Returns total lectures conducted for a subject
    int totalLecturesConducted(String subject) {
        int total = 0;
        for (List<LocalDate> dates : lecturesConducted.get(subject).values()) {
            total += dates.size();
        }
        return total;
    }

    // Returns overall attendance percentage across all subjects
    double overallAttendancePercentage(List<String> subjects) {
        int totalAttended = 0;
        int totalConducted = 0;
        for (String subject : subjects) {
            totalAttended += totalLecturesAttended(subject);
            totalConducted += totalLecturesConducted(subject);
        }
        if (totalConducted == 0) return 0;
        return ((double) totalAttended / totalConducted) * 100;
    }
}

public class AttendanceSystem {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        List<String> subjectsList = Arrays.asList("Maths", "Science", "English", "Hindi", "Marathi", "Geography");
        ArrayList<Student> studentList = new ArrayList<>();

        // Pre-added students
        studentList.add(new Student("Amit", 11, subjectsList));
        studentList.add(new Student("Ravi", 12, subjectsList));
        studentList.add(new Student("Neha", 13, subjectsList));
        studentList.add(new Student("Priya", 14, subjectsList));
        studentList.add(new Student("Karan", 15, subjectsList));

        boolean continueProgram = true;

        while (continueProgram) {
            int userChoice = 0;

            // Display menu and get valid input
            while (true) {
                System.out.println("\n========= Attendance Management System =========");
                System.out.println("1. Register New Student");
                System.out.println("2. Show Attendance of a Student");
                System.out.println("3. Mark Attendance");
                System.out.println("4. Exit");
                System.out.print("Enter your choice (1-4): ");
                if (scanner.hasNextInt()) {
                    userChoice = scanner.nextInt();
                    scanner.nextLine();
                    if (userChoice >= 1 && userChoice <= 4) break;
                    else System.out.println("Invalid choice! Please select 1-4.");
                } else {
                    System.out.println("Invalid input! Please enter a number.");
                    scanner.nextLine();
                }
            }

            switch (userChoice) {
                case 1: // Register new student
                    System.out.print("Enter student name: ");
                    String name = scanner.nextLine();

                    int roll = 0;
                    while (true) {
                        System.out.print("Enter student roll number: ");
                        if (scanner.hasNextInt()) {
                            roll = scanner.nextInt();
                            scanner.nextLine();

                            boolean exists = false;
                            for (Student s : studentList) {
                                if (s.rollNumber == roll) {
                                    exists = true;
                                    break;
                                }
                            }
                            if (exists) System.out.println("Roll number already exists! Enter a unique roll number.");
                            else break;

                        } else {
                            System.out.println("Invalid input! Please enter a valid roll number.");
                            scanner.nextLine();
                        }
                    }

                    Student newStudent = new Student(name, roll, subjectsList);
                    // Copy existing lectures for new student
                    for (Student s : studentList) {
                        for (String sub : subjectsList) {
                            for (Map.Entry<Integer, List<LocalDate>> entry : s.lecturesConducted.get(sub).entrySet()) {
                                int lectureNo = entry.getKey();
                                List<LocalDate> dates = entry.getValue();
                                newStudent.lecturesConducted.get(sub).put(lectureNo, new ArrayList<>(dates));
                            }
                        }
                    }
                    studentList.add(newStudent);
                    System.out.println("Student registered successfully!");
                    break;

                case 2: // Show attendance
                    System.out.print("Enter student name: ");
                    String searchName = scanner.nextLine();
                    int searchRoll = 0;
                    while (true) {
                        System.out.print("Enter student roll number: ");
                        if (scanner.hasNextInt()) {
                            searchRoll = scanner.nextInt();
                            scanner.nextLine();
                            break;
                        } else {
                            System.out.println("Invalid input! Please enter a number.");
                            scanner.nextLine();
                        }
                    }

                    boolean studentFound = false;
                    for (Student s : studentList) {
                        if (s.studentName.equalsIgnoreCase(searchName) && s.rollNumber == searchRoll) {
                            System.out.println("\n--- Attendance Report ---");
                            System.out.println("Name: " + s.studentName + ", Roll No: " + s.rollNumber);

                            for (String sub : subjectsList) {
                                int conducted = s.totalLecturesConducted(sub);
                                int attended = s.totalLecturesAttended(sub);
                                double percent = conducted == 0 ? 0 : ((double) attended / conducted) * 100;
                                System.out.printf("%s: %d/%d lectures attended (%.2f%%)\n", sub, attended, conducted, percent);
                            }

                            System.out.printf("Overall Attendance: %.2f%%\n", s.overallAttendancePercentage(subjectsList));
                            studentFound = true;
                            break;
                        }
                    }
                    if (!studentFound) System.out.println("Student is not present!");
                    break;

                case 3: // Mark attendance
                    int lectureNo = 0;
                    while (true) {
                        System.out.print("Enter Lecture Number: ");
                        if (scanner.hasNextInt()) {
                            lectureNo = scanner.nextInt();
                            scanner.nextLine();
                            if (lectureNo <= 0) System.out.println("Lecture number must be positive!");
                            else break;
                        } else {
                            System.out.println("Invalid input! Enter a number.");
                            scanner.nextLine();
                        }
                    }

                    System.out.println("\nSelect Subject for this Lecture:");
                    for (int i = 0; i < subjectsList.size(); i++) {
                        System.out.println((i + 1) + ". " + subjectsList.get(i));
                    }

                    int subjectChoice = 0;
                    String subjectSelected = "";
                    while (true) {
                        System.out.print("Enter subject number: ");
                        if (scanner.hasNextInt()) {
                            subjectChoice = scanner.nextInt();
                            scanner.nextLine();
                            if (subjectChoice >= 1 && subjectChoice <= subjectsList.size()) {
                                subjectSelected = subjectsList.get(subjectChoice - 1);
                                break;
                            } else {
                                System.out.println("Invalid choice! Please select valid subject.");
                            }
                        } else {
                            System.out.println("Invalid input! Enter a number.");
                            scanner.nextLine();
                        }
                    }

                    // Prevent duplicate lecture number for the same subject
                    boolean lectureExists = false;
                    for (Student s : studentList) {
                        if (s.lecturesConducted.get(subjectSelected).containsKey(lectureNo)) {
                            lectureExists = true;
                            break;
                        }
                    }

                    if (lectureExists) {
                        System.out.println("Attendance already entered for " + subjectSelected + " Lecture " + lectureNo + ". Choose a different lecture number.");
                        break;
                    }

                    LocalDate today = LocalDate.now();
                    System.out.println("\nMarking attendance for " + subjectSelected + " Lecture " + lectureNo + " on " + today + ":");
                    for (Student s : studentList) {
                        String presentInput;
                        while (true) {
                            System.out.print("Is " + s.studentName + " (Roll No: " + s.rollNumber + ") present? (y/n): ");
                            presentInput = scanner.nextLine().trim().toLowerCase();
                            if (presentInput.equals("y") || presentInput.equals("n")) break;
                            else System.out.println("Invalid input! Enter 'y' or 'n'.");
                        }
                        s.markAttendance(subjectSelected, lectureNo, presentInput.equals("y"), today);
                    }

                    System.out.println("Attendance recorded for " + subjectSelected + " Lecture " + lectureNo + " on " + today + "!");
                    break;

                case 4: // Exit
                    System.out.println("Exiting program. Goodbye!");
                    continueProgram = false;
                    break;
            }
        }

        scanner.close();
    }
}