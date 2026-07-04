
LAB WORKBENCH: SELF-SERVICE PASSWORD RESET (SSPR) GATE
================================================================================

1. PRE-DEPLOYMENT TARGET IDENTIFICATION
--------------------------------------------------------------------------------
To test this scoped access gate accurately, we will utilize a dedicated pilot group 
container alongside our standard directory test user profiles within Microsoft 
Entra ID (MEID):

* Target Pilot Group: SSPR-Pilot-Group (Static Security Group)
* Alpha Engineer (Source): Added as a direct member of SSPR-Pilot-Group to verify 
  successful Self-Service Password Reset (SSPR) authorization.
* Bravo Engineer (Source): Explicitly left OUT of SSPR-Pilot-Group to verify the 
  hard scoped block behavior of the portal gateway.

2. COMPONENT DEPLOYMENT & PORTAL PATHS
--------------------------------------------------------------------------------
Execute the directory configurations precisely using the following steps within 
the Microsoft Entra admin center:

Step 1: Create the Scoped Pilot Container
  - Navigate to Microsoft Entra ID (MEID) > Groups > All groups > New group.
  - Set Group type to "Security", and Name to "SSPR-Pilot-Group".
  - Click "Members", search for and select Alpha Engineer (Source).
  - Click "Create".

tep 2: Navigate to the Global SSPR Properties Blade
Step 2a: Look at the very top left of your screen at the breadcrumb trail (the horizontal navigation path). Click on Microsoft Entra ID to jump completely out of the group and back to the root tenant overview page.

Step 2b: Look at the main left-hand navigation column. Scroll down until you see the Identity category, expand it, and then look for a sub-category named Protection. Expand Protection.

Step 2c: Inside that expanded Protection list, click directly on Password reset.

Step 2d: You will land directly on the global Properties blade. This is where you will see the exact setting we need: "Self service password reset enabled" with the options None, Selected, or All.

Step 3: Configure Authentication Methods Gate
  - In the left menu under the "Password reset" header, click "Authentication methods".
  - Set "Number of methods required to reset" to "2".
  - Under "Methods available to users", check the boxes for:
    * "Mobile app notification"
    * "Mobile app code"
  - Uncheck all other methods (Email, Mobile phone, etc.) to enforce strict 
    Multi-Factor Authentication (MFA) application compliance.
  - Click "Save" at the top of the blade.

3. EXPECTED OPERATIONAL HANDSHAKE & INVERSE LOGIC
--------------------------------------------------------------------------------
The MEID password recovery engine applies a strict boolean evaluation gate based 
on group scope checking at the initial entry point:

* The Failure State (Bravo Reset Blocked): Bravo Engineer navigates to the standard 
  password recovery landing portal (https://passwordreset.microsoftonline.com). 
  Bravo inputs their User Principal Name. The SSPR engine validates the identity 
  against the targeted configuration scope. Because Bravo is not a member of 
  SSPR-Pilot-Group, the engine returns a hard boolean negative state, completely 
  blocks access to the verification screen, and displays an error stating that the 
  account is not enabled for password reset.

* The Success State (Alpha Reset Allowed): Alpha Engineer navigates to the password 
  recovery landing portal and inputs their User Principal Name. The SSPR engine 
  validates that Alpha exists inside SSPR-Pilot-Group. The engine returns a hard 
  boolean positive state, unlocks the next phase of the pipeline, and prompts Alpha 
  to satisfy the two mandatory mobile application MFA challenge barriers before 
  permitting the directory write modification.

4. LIVE VERIFICATION EXECUTION PLAN
--------------------------------------------------------------------------------
Execute these verification steps immediately after allowing 3-5 minutes for 
tenant-wide policy replication:

1. Open a fresh private Incognito browser window.
2. Navigate directly to the recovery URL: https://passwordreset.microsoftonline.com
3. Input Bravo Engineer's User Principal Name, complete the captcha, and click Next.  
4. Verify the gateway throws an immediate block message indicating "You cannot 
   reset your own password because password reset isn't turned on for your account."
5. Close the window, open a new private window, and navigate back to the recovery URL.
6. Input Alpha Engineer's User Principal Name, complete the captcha, and click Next.
7. Verify that Alpha cleanly bypasses the initial block page and successfully lands 
   on the mobile app authentication challenge screen.
================================================================================

================================================================================
LAB COMPLETION SUMMARY: SELF-SERVICE PASSWORD RESET (SSPR) GATE
================================================================================

1. DIRECTORY CONTAINER AND TARGET SETUP
--------------------------------------------------------------------------------
To control the scope of the self-remediation pipeline, a dedicated boundary group 
was established within Microsoft Entra ID (MEID):

* Static Pilot Group: SSPR-Pilot-Group
* Alpha Engineer (Source): Manually added as a direct member of SSPR-Pilot-Group. 
  Alpha represents the authorized remediation pathway.
* Bravo Engineer (Source): Explicitly omitted from SSPR-Pilot-Group. Bravo 
  represents the unauthorized/unscoped pathway.

2. SYSTEM CONVERGENCE CONFIGURATION
--------------------------------------------------------------------------------
Two integrated configuration blades were updated inside the Entra ID admin portal 
to drop the security gate:

* SSPR Properties Blade:
  - Configuration Path: Entra ID > Password reset > Properties.
  - Enforcement Toggle: Set to "Selected".
  - Scope Assignment: Bound explicitly to "SSPR-Pilot-Group".

* Global Authentication Methods Policy:
  - Configuration Path: Entra ID > Authentication methods > Policies > Microsoft Authenticator.
  - Target Assignment: Configured to target the specific group "SSPR-Pilot-Group".
  - SSPR Requirement: Set to "2" methods, enforcing both Mobile App Notification 
    and Mobile App Code verification constraints.

3. WHY THIS CONFIGURATION WORKS
--------------------------------------------------------------------------------
The MEID identity engine evaluates self-service recovery tracking requests at the 
absolute front door before exposing authentication challenges:

* The Failure State (Bravo Blocked): When Bravo Engineer (Source) hits the portal, 
  the engine runs a policy check against the "Password reset" properties list. 
  Because Bravo's identity is missing from SSPR-Pilot-Group, the engine returns a 
  hard boolean negative. The portal immediately cuts the session, completely 
  blocking access to the recovery pipeline and throwing an explicit "password 
  reset isn’t turned on for your account" error.

* The Success State (Alpha Allowed): When Alpha Engineer (Source) hits the portal, 
  the engine verifies that Alpha belongs to SSPR-Pilot-Group. The block gate lifts, 
  and the engine passes the session to the second phase. Alpha is permitted to satisfy 
  the mandatory two-step mobile application MFA validation steps, resulting in a 
  successful write-back password modification.
================================================================================
