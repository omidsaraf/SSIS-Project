To show code in your GitHub repository with proper formatting and syntax highlighting, you can use Markdown. Here's a step-by-step guide on how to do it:

### Step-by-Step Guide to Show Code in GitHub

1. **Create or Edit a Markdown File**:
   - Navigate to your repository on GitHub.
   - Click on the `Add file` button and select `Create new file`, or navigate to an existing Markdown file (`.md`) that you want to edit.

2. **Use Fenced Code Blocks**:
   - To add code with syntax highlighting, use fenced code blocks. These are created by placing triple backticks (```) before and after your code block.
   - Optionally, you can specify the language for syntax highlighting by adding the language name after the opening backticks.

### Example

Here’s an example of how to format a PowerShell script in a Markdown file:

```markdown
# Security Patching Job

This script downloads and applies the latest security patches to SQL Server.

```powershell
# Step 1: Download the latest updates
$updates = Get-WindowsUpdate -MicrosoftUpdate -AcceptAll -Install

# Step 2: Apply the updates
Install-WindowsUpdate -MicrosoftUpdate -AcceptAll -Install

# Step 3: Restart the SQL Server service if necessary
Restart-Service -Name 'MSSQLSERVER' -Force

# Step 4: Send an email notification
Send-MailMessage -To 'admin@example.com' -From 'sqlserver@example.com' -Subject 'SQL Server Security Patching Completed' -Body 'The latest security patches and updates have been successfully applied to the SQL Server.' -SmtpServer 'smtp.example.com'
```
```

3. **Commit Your Changes**:
   - After adding your code, scroll down to the bottom of the page.
   - Enter a commit message describing your changes.
   - Click on the `Commit new file` button to save your changes.

### Viewing the Code

Once committed, your code will be displayed with proper formatting and syntax highlighting when viewed in the GitHub repository.

### Additional Tips

- **Inline Code**: For inline code snippets, use single backticks. For example, `SELECT * FROM Users`.
- **Syntax Highlighting**: GitHub supports many programming languages for syntax highlighting. You can find a list of supported languages in the [GitHub documentation](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks).

By following these steps, you can ensure that your code is clearly presented and easy to read in your GitHub repository. If you have any more questions or need further assistance, feel free to ask!
