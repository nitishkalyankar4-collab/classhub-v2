# ClassHub V2 - Quick Start Guide 🚀

## **5-Minute Setup**

### Step 1: Open the App (30 seconds)
```bash
# Option A: Direct browser open
open index.html  (macOS)
xdg-open index.html  (Linux)
start index.html  (Windows)

# Option B: Use any web server
python -m http.server 8000
# Visit: http://localhost:8000
```

### Step 2: Login (1 minute)

Choose a role and login:

| Role | Email | Password | Use Case |
|------|-------|----------|----------|
| **Admin** | admin@school.com | admin123 | Manage school |
| **Teacher** | teacher@school.com | teacher123 | Create classes |
| **Student** | student@school.com | student123 | Learn |
| **Parent** | parent@school.com | parent123 | Monitor child |

### Step 3: Explore (3-4 minutes)

**As Teacher:**
1. Dashboard → "+ Create Class"
2. Enter name: "Class 10 Math"
3. Fill subject, section, description
4. 🎉 Classroom created with code!

**As Student:**
1. Dashboard → "+ Join Class"
2. Enter join code
3. 🎉 Classroom joined!

**As Parent:**
1. Dashboard → "Link Child's Account"
2. Enter child's email
3. 🎉 Request sent!

**As Admin:**
1. View all statistics
2. Add teachers
3. Monitor school activity

---

## **Key Features to Try**

### 📚 Smart Tutor (AI)
- Click "Smart Tutor" in sidebar
- Ask: "What is photosynthesis?"
- Get instant AI explanation
- Ask follow-ups

### 📚 Assignments
- In classroom → Assignments tab
- Teacher: Click "+ Create"
- Student: Click assignment to submit

### 📚 Study Notes
- In classroom → Notes tab
- Teacher: Upload PDF or type notes
- Student: Download for offline access

### 📋 Classroom Chat
- In classroom → Chat tab
- Send messages to class
- Reply to discussions
- See message history

### 👤 User Profiles
- Click "My Profile" in sidebar
- Fill education/qualification info
- Save preferences
- Upload profile picture

### ⚙️ Settings
- Click "Settings" in sidebar
- Toggle notifications
- Change password
- Set preferences

---

## **Common Tasks**

### Create Assignment (Teacher)
```
1. Classroom → Assignments tab
2. Click "+ Create"
3. Title: "Chapter 5 Problems"
4. Due date: Tomorrow
5. Click "Create"
```

### Submit Assignment (Student)
```
1. Dashboard → Pending Assignments
2. Click assignment
3. Upload file or type answer
4. Click "Submit"
```

### Share Classroom
```
1. Classroom page shows join code
2. Copy: ABC123
3. Share via WhatsApp, email, QR code
4. Students enter code to join
```

### Use AI Tutor
```
1. Sidebar → Smart Tutor
2. Ask any academic question
3. Get AI explanation
4. Ask follow-up questions
5. Copy answers to notes
```

### Grade Assignment (Teacher)
```
1. Classroom → Assignments
2. View submissions
3. Click assignment
4. Give grade and feedback
5. Student notified
```

---

## **Tips & Tricks**

💡 **Data Persistence**: Everything saves automatically to browser  
💡 **Multi-classroom**: Teachers can create multiple classes  
💡 **Mobile Ready**: Works perfectly on phones  
💡 **No Backend**: Works completely client-side  
💡 **Share Easily**: Give join code to students (no passwords)  
💡 **Search Feature**: Search messages, notes, assignments (top bar)  
💡 **Notifications**: Red badge shows unread count  
💡 **Offline Access**: Download notes for offline reading  

---

## **Troubleshooting**

**Q: Data not saving?**  
A: Check if localStorage is enabled. Settings → Privacy → Cookies  

**Q: Can't join classroom?**  
A: Verify join code is correct (6 chars, uppercase)  

**Q: AI not responding?**  
A: Check internet connection. AI needs connectivity.  

**Q: Profile not updating?**  
A: Click "Save Profile" button. Refresh page if needed.  

---

## **Next Steps**

1. 📖 Read full [README.md](README.md) for detailed features
2. 💪 Customize colors in CSS variables
3. 🔧 Add your school name in admin settings
4. 🚀 Deploy to GitHub Pages for permanent URL
5. 💰 Set up backend database (production)
6. 💋 Get Gemini API key for Smart Tutor
7. 👥 Invite real users to your instance

---

**🌟 That's it! You're ready to go. Enjoy ClassHub!**

Need help? Check README.md or GitHub Issues.
