

### App concept

What it does: allows users upload materials and shows them.
For whom: students of NewUU.
The single most important thing a user can do: find materials.

### Main user flow

1. Sign up screen.
	- Data: email, major, current year.
		- What if user lies?
		- We make changing major/year only possible through contacting an admin.
	- Creates pwd with their `*@newuu.uz` email as main credential.
	- During processing the request, the server emails one time code to the user for confirmation.
3. Confirmation screen.
4. Main home page.
	- Note: only materials in their major is shown.
	- Filters: academic year, student year, course.
		- Why: because despite having the same major, courses or their order may change.
	- Shows materials grouped by material type (image, text-message, link, etc.)
5. Links to material detail page (where they can download).
6. Or from (5), they can navigate to pages in the drawer:
	- Home
	- Upload
	- Account
	- Downloads
	- My uploads
7. Upload page flow:
	- Constraints:
		- Can upload only for the student's major.
	- Upload new material with necessary info.
		- Info: academic year, course name.
		- onSuccess: return to my uploads.
	- Click send (status becomes pending).
8. Account:
	- Delete
	- Edit info
		- Opens page to email form
			- EMAIL is not editable.
			- email form in a sense that it triggers backend to send message to admin.
	- Change password
		- Opens page to change pwd form.
			- Send form: server sends confirmation code.
				- Opens confirm change pwd screen.
					- onSuccess: return to Account page.
9. Downloads:
	- shows: filters: academic year, course
	- initial filter: none (shows all downloads)
		- enhancement: group by resource type
			- link/image/pdf/etc.
	- feature: multi-select (ex use case: delete multiple docs)
	- each card clickable
		- has delete from downloads button
		- onclick: material detail
		- materialDetail: has delete from downloads button
10. My uploads.
	- Accepted
		- show: list of materials
			- each card: clickable
				- onClick: open material detail
	- Pending (to be reviewed by Admin)
		- show: list of materials
			- each card: clickable
				- onClick: open materials detail
	-  Rejected
		- The rejected material is stored on the user's device in cache and cleaned up periodically
			- BUT: allow deleting manually as in `Downloads` (multi-select, delete on each card, delete on material detail)
		- same view as in previous two except: includes reject status + text on when this material will be deleted from user's device	
	- Upload new button

### Screen list and wireframes

- Just infer screens and create wireframes based on `Main user flow`

# Flutter plan

Note/Instruction: AI generates this plan based on `Main user flow`. No need to be precise, as this is just a proposal. Riverpod is used for state management. Gorouter is used for routing.

Example for AI:
- Home screen:
	- Tree:
```
	Column
	|    |_-------------------------------+
Row (filters)                    Column (or similar widget 
									[even custom])
		|                                  |
children:                                  |____+
	Container with filter label               Child 1: label of material type
	and some widget for dropdown              Child2: list view with tiles (or 
	(ex: DropdownButton from material                  yet another column with 
	lib)                                               custom `material` card)

```

## Data model

```

class User {
	final String email;
	final String major;
	final int year;
	final DateTime dateCreated;
	
	User ({ required this.email, required this.major, required this.year });
}

enum Status {
	Pending,
	Rejected,
	Accepted
}

class Course {
	final String name;
	final String major;
	final String academicYear;
	final DateTime createdAt;
}

class Material {
	final String courseId;
	final String major;
	final String academicYear; /// ex: 2026/2027 fall
	final String type; //  enough
	final String uploadedByUserEmail;
	final DateTime uploadedAt;
	
	Material(.....);
}

/// NOTE: we may create classes for forms and in-app data sharing. This
///       these are details and are not part of Proposla which is just a
///       rough sketch. Also, we can think about optimization.

```

### Out of scope

- Backend
- Organic community effort -- admin + voting mechanism
	- Explanation:
		- In this desitgn, users upload and attache major, academic year, and course name.
		  Users then vote/downvote for this upload. Higher votes mean legitimacy.
		  On reaching some threshold:
			- if votes > downvote and votes > threshold:
				- auto-add material to permanent (if course does not exist in db, add it)
				- Enhancement: small fast gen-ai to normalize the inputs (also prevent rogue uploads by analyzing material content and estimating fit)
		  Admin exists to review and prevent falsely (rogue)  roguely voted accept.

