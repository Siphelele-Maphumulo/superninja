# superninja


<%@page import="java.time.LocalTime"%>
<%@page import="myPackage.*"%>
<%@page import="java.util.*"%>
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<jsp:useBean id="pDAO" class="myPackage.DatabaseClass" scope="page"/>

<%
if (request.getParameter("page").toString().equals("login")) {
    String userName = request.getParameter("username").toString();
    String userPass = request.getParameter("password").toString();

    if (pDAO.loginValidate(userName, userPass)) {
        session.setAttribute("userStatus", "1");
        session.setAttribute("userId", pDAO.getUserId(userName));
        response.sendRedirect("dashboard.jsp");
    } else {
        request.getSession().setAttribute("userStatus", "-1");
        response.sendRedirect("login.jsp");
    }

} else if (request.getParameter("page").toString().equals("register")) {
    String fName = request.getParameter("fname");
    String lName = request.getParameter("lname");
    String uName = request.getParameter("uname");  // This is the staffNum (MUT Identity Number)
    String email = request.getParameter("email");
    String pass = request.getParameter("pass");
    String contactNo = request.getParameter("contactno");
    String city = request.getParameter("city");
    String address = request.getParameter("address");
    
    // Hash the password using BCrypt before saving to the database
    String hashedPass = PasswordUtils.bcryptHashPassword(pass);
    System.out.println("My hashed pass: "+ hashedPass);

    // Check if the lecturer's email exists
    boolean lecturerExists = pDAO.checkLecturerByEmail(email);
    
    // Log to console whether it exists or not
    if (lecturerExists) {
        System.out.println("Lecturer with email " + email + " exists.");
    } else {
        System.out.println("Lecturer with email " + email + " does not exist.");
    }
    
    // Proceed with student registration using the hashed password
    pDAO.addNewStudent(fName, lName, uName, email, hashedPass, contactNo, city, address);

    // Redirect to login.jsp after successful registration
    response.sendRedirect("login.jsp");
}

else if(request.getParameter("page").toString().equals("profile")){
        
        String fName =request.getParameter("fname");
        String lName =request.getParameter("lname");
        String uName=request.getParameter("uname");
        String email=request.getParameter("email");
        String pass=request.getParameter("pass");
        String contactNo =request.getParameter("contactno");
        String city =request.getParameter("city");
        String address =request.getParameter("address");
         String uType =request.getParameter("utype");
        int uid=Integer.parseInt(session.getAttribute("userId").toString());
    
         
    pDAO.updateStudent(uid,fName,lName,uName,email,pass,contactNo,city,address,uType);
    response.sendRedirect("dashboard.jsp");
}else if(request.getParameter("page").toString().equals("courses")) {
    if(request.getParameter("operation").toString().equals("addnew")) {
        String courseName = request.getParameter("coursename");
        int totalMarks = Integer.parseInt(request.getParameter("totalmarks"));
        String time = request.getParameter("time");
        String examDate = request.getParameter("examdate"); // New exam date parameter

        // Pass the examDate to the addNewCourse method
        pDAO.addNewCourse(courseName, totalMarks, time, examDate);
        
        response.sendRedirect("adm-page.jsp?pgprt=2");
    } else if(request.getParameter("operation").toString().equals("del")) {
        pDAO.delCourse(request.getParameter("cname").toString());
        response.sendRedirect("adm-page.jsp?pgprt=2");
    }
}
else if(request.getParameter("page").toString().equals("accounts")){
    if(request.getParameter("operation").toString().equals("del")){
        pDAO.delUser(Integer.parseInt(request.getParameter("uid")));
        response.sendRedirect("adm-page.jsp?pgprt=1");
    }

}

else if (request.getParameter("page").toString().equals("questions")) {
    if (request.getParameter("operation").toString().equals("addnew")) {
        String courseName = request.getParameter("coursename"); // Retrieve the course name from the form
        String questionType = request.getParameter("questionType");

        // Ensure course name is not null or empty
        if (courseName != null && !courseName.trim().isEmpty()) {
            // Pass the question type along with other parameters to DatabaseClass
            pDAO.addQuestion(
                courseName, // Pass courseName here
                request.getParameter("question"),
                request.getParameter("opt1"),
                request.getParameter("opt2"),
                request.getParameter("opt3"),
                request.getParameter("opt4"),
                request.getParameter("correct"),
                questionType  // The new question type parameter
            );
            
            // Redirect back to the questions page after insertion
            response.sendRedirect("adm-page.jsp?pgprt=3");
        } else {
            out.println("Error: Course name cannot be empty.");
        }
    }
}


else if(request.getParameter("page").toString().equals("exams")){
    if(request.getParameter("operation").toString().equals("startexam")){
        String cName=request.getParameter("coursename");
        int userId=Integer.parseInt(session.getAttribute("userId").toString());
        
        int examId=pDAO.startExam(cName,userId);
        session.setAttribute("examId",examId);
        session.setAttribute("examStarted","1");
        response.sendRedirect("std-page.jsp?pgprt=1&coursename="+cName);
    }else if(request.getParameter("operation").toString().equals("submitted")){
        try{
        String time=LocalTime.now().toString();
        int size=Integer.parseInt(request.getParameter("size"));
        int eId=Integer.parseInt(session.getAttribute("examId").toString());
        int tMarks=Integer.parseInt(request.getParameter("totalmarks"));
        session.removeAttribute("examId");
        session.removeAttribute("examStarted");
        for(int i=0;i<size;i++){
            String question=request.getParameter("question"+i);
            String ans=request.getParameter("ans"+i);
            
            int qid=Integer.parseInt(request.getParameter("qid"+i));
            
            pDAO.insertAnswer(eId,qid,question,ans);
        }
        System.out.println(tMarks+" conn\t Size: "+size);
        pDAO.calculateResult(eId,tMarks,time,size);
        
        response.sendRedirect("std-page.jsp?pgprt=1&eid="+eId+"&showresult=1");
        }catch(Exception e){
            e.printStackTrace();
        }
        
        
    }

}else if(request.getParameter("page").toString().equals("logout")){
    session.setAttribute("userStatus","0");
    session.removeAttribute("examId");
    session.removeAttribute("examStarted");
    response.sendRedirect("index.jsp");
}

%>
