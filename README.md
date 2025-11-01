import java.io.*;
import java.sql.*;
import jakarta.servlet.*;
import jakarta.servlet.http.*;

public class MainServlet extends HttpServlet {
    private static final String URL = "jdbc:mysql://localhost:3306/testdb";
    private static final String USER = "root";
    private static final String PASS = "password";

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();

        String action = request.getParameter("action");

        // ---------- LOGIN PAGE ----------
        if (action == null) {
            out.println("<h2>Welcome</h2>");
            out.println("<h3>Choose an option:</h3>");
            out.println("<a href='MainServlet?action=loginform'>Login</a><br>");
            out.println("<a href='MainServlet?action=employees'>View Employees</a><br>");
            out.println("<a href='MainServlet?action=attendanceform'>Student Attendance</a><br>");
        }

        // ---------- LOGIN FORM ----------
        else if (action.equals("loginform")) {
            out.println("<h2>User Login</h2>");
            out.println("<form action='MainServlet?action=login' method='post'>");
            out.println("Username: <input type='text' name='username'><br><br>");
            out.println("Password: <input type='password' name='password'><br><br>");
            out.println("<input type='submit' value='Login'>");
            out.println("</form>");
        }

        // ---------- EMPLOYEE LIST ----------
        else if (action.equals("employees")) {
            out.println("<h2>Employee Records</h2>");
            out.println("<form method='get' action='MainServlet'>");
            out.println("<input type='hidden' name='action' value='employees'>");
            out.println("Search by EmpID: <input type='text' name='empid'>");
            out.println("<input type='submit' value='Search'></form><br>");

            String empId = request.getParameter("empid");

            try (Connection con = DriverManager.getConnection(URL, USER, PASS);
                 Statement st = con.createStatement()) {
                String sql = (empId != null && !empId.isEmpty())
                        ? "SELECT * FROM Employee WHERE EmpID=" + empId
                        : "SELECT * FROM Employee";

                ResultSet rs = st.executeQuery(sql);
                out.println("<table border='1'><tr><th>EmpID</th><th>Name</th><th>Salary</th></tr>");
                while (rs.next()) {
                    out.println("<tr><td>" + rs.getInt("EmpID") + "</td><td>" +
                            rs.getString("Name") + "</td><td>" +
                            rs.getDouble("Salary") + "</td></tr>");
                }
                out.println("</table>");
            } catch (Exception e) {
                out.println("<p>Error: " + e.getMessage() + "</p>");
            }
            out.println("<br><a href='MainServlet'>Home</a>");
        }

        // ---------- ATTENDANCE FORM ----------
        else if (action.equals("attendanceform")) {
            out.println("<h2>Student Attendance Form</h2>");
            out.println("<form action='MainServlet?action=saveattendance' method='post'>");
            out.println("Student ID: <input type='text' name='studentid'><br><br>");
            out.println("Date: <input type='date' name='date'><br><br>");
            out.println("Status: <select name='status'>");
            out.println("<option value='Present'>Present</option>");
            out.println("<option value='Absent'>Absent</option>");
            out.println("</select><br><br>");
            out.println("<input type='submit' value='Submit Attendance'>");
            out.println("</form>");
        }
    }

    // ---------- HANDLE POST REQUESTS ----------
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        String action = request.getParameter("action");

        // ---------- LOGIN VALIDATION ----------
        if ("login".equals(action)) {
            String user = request.getParameter("username");
            String pass = request.getParameter("password");

            if ("admin".equals(user) && "12345".equals(pass)) {
                out.println("<h2>Welcome, " + user + "!</h2>");
            } else {
                out.println("<h3>Invalid credentials. Please try again.</h3>");
                out.println("<a href='MainServlet?action=loginform'>Back</a>");
            }
        }

        // ---------- SAVE ATTENDANCE ----------
        else if ("saveattendance".equals(action)) {
            int sid = Integer.parseInt(request.getParameter("studentid"));
            String date = request.getParameter("date");
            String status = request.getParameter("status");

            try (Connection con = DriverManager.getConnection(URL, USER, PASS);
                 PreparedStatement ps = con.prepareStatement(
                         "INSERT INTO Attendance(StudentID, Date, Status) VALUES (?, ?, ?)")) {
                ps.setInt(1, sid);
                ps.setString(2, date);
                ps.setString(3, status);
                ps.executeUpdate();
                out.println("<h3>Attendance Saved Successfully!</h3>");
            } catch (Exception e) {
                out.println("<p>Error: " + e.getMessage() + "</p>");
            }
            out.println("<a href='MainServlet?action=attendanceform'>Back</a>");
        }
    }
}
