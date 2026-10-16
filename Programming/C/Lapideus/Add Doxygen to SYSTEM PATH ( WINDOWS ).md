l

Great! Since Doxygen is installed in `C:\Program Files\doxygen`, but Windows **doesn’t recognize it as a command**, you need to **add it to the system PATH**.

---

### **🔹 How to Add Doxygen to the System PATH (Windows 10/11)**

1️⃣ **Open Environment Variables:**

- Press **`Win + R`**, type **`sysdm.cpl`**, and press **Enter**.
- Go to the **Advanced** tab → Click **Environment Variables**.

2️⃣ **Find the "Path" Variable:**

- Under **System Variables**, find **Path** and click **Edit**.

3️⃣ **Add the Doxygen Path:**

- Click **New** and add:
    
    ```
    C:\Program Files\doxygen\bin
    ```
    
- Click **OK** to save changes.

4️⃣ **Restart Command Prompt:**

- Close and reopen **cmd** or restart your PC.
- Run this command to verify:
    
    ```sh
    doxygen -v
    ```
    
    If it shows a version number, Doxygen is now working globally! 🎉

---

### **🔹 Now You Can Run Doxygen Anywhere!**

- Navigate to your project directory:
    
    ```sh
    cd C:\path\to\your\project
    ```
    
- Run Doxygen:
    
    ```sh
    doxygen Doxyfile
    ```
    

Would you like help generating your first documentation with Doxygen? 🚀