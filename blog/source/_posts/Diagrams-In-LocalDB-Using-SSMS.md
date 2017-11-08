title: Diagrams In LocalDB Using SSMS
layout: post
tags:
  - ASP.NET Core MVC
  - SQL Server
  - SQL Server Management Studio
  - Entity Framework
categories:
  - Development
  - Database
authorId: james_chambers
featureImage: logo_579.png
originalUrl: 'http://jameschambers.com/'
date: 2017-11-06 15:17:28
---

Often you'll find that a project is of a size where the data model is easier to visualize if you...well...if you visualize it. The canonical way to do this for those of us on the MS stack is through database diagrams in SQL Server Management Studio.

![Make Sense of Your LocalDb Models with SSMS](https://jcblogimages.blob.core.windows.net/img/2017/ssms-title-slide.png)

<!-- more -->

## Not Every Project Has a Data Analyst

Historically, a large number of organizations have worked with data analysts that develop the schema for an application given a set of requirements using a set of powerful designer tools. These tools are capable of generating change scripts, comparing databases for changes and drawing out the relationships between entities. This is more true for larger applications, certainly more true in the enterprise space, and is a practice you can still see in operation today particularily in working with legacy code.

However, for smaller shops and projects and in the agile space, this is not the common approach. Many folks choose to evolve data models on the fly, use tools like data projects or migration tools like Roundhouse. And, in today's advancements in EF, there are many projects that elect to use code first migrations which have evolved far beyond the one-way, strongly opinionated roots that first came to Entity Framework back in the unicorn days.

In my travels and in mentoring developers over the last few years I've found a surprising number of folks who haven't seen the tooling that is freely available to them through SQL Server Management Studio, also known as SSMS.

## So, What Does My Data Look Like?

In an effort to better understand a fix or feature that I'm asked to help with, I often will sketch out the problem and the implementation. When this includes modifications to the schema I like to employ the use of a diagram in my effort.  While pen-and-paper would do just fine for most cases, it's often handy to know data types, see any relationships and move things around as often as needed to clarify the picture of the problem I'm trying to solve.  And this is where tooling comes into play.

SSMS is a medium-weight utility that clocks in just under a gig ([you can download it here](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)). It's by no means small, but it is powerful and has some great cloud-connectivity capabilities as well. It's designed for the fully-featured SQL Server SKUs, but it works just fine for LocalDB as well.

To get started with diagrams in LocalDB, simply invoke the context menu with a right-click on your database's "Database Diagrams" node in SSMS's Object Explorer.

![Context Menu When Clicking on a Database in the SSMS Object Explorer](https://jcblogimages.blob.core.windows.net/img/2017/ssms-db-context-menu.png)

Click on the option for "Install Diagram Support".

***Note*** I run on Windows 10 and I am not on a domain. There is no active directory on my machine. I have found that when I use code-first databases and EF migrations that I will get an error like `Microsoft SQL Server Error: 15404` saying:

>LocalDb could not obtain information about group/user (username)).

This error also surfaces as 'SQL Server error 0x534', '0x54b', '0x2147'.

You can get past this error by elevating your privileges on the database in question using the following command. To execute, just right-click on your database to get that context menu back and select "New Query", then paste this in (and fix your DB name):

`alter authorization on database::[your-db-name-no-quotes] to sa`

This is okay with LocalDb instances running on your machine, as I'm assuming that you're going to be running this locally. Be sure to consider best practicies, application-level security and your company policies before changing database permissions for databases your organization makes available at large. 

With security set and ready to go you're pretty much off to the races.

## Creating Diagrams

This part is super easy.  Now that diagram support has been added just invoke that context menu again, but instead, this time choose "New Database Diagram".  A blank canvas will appear and you'll be prompted to add tables.

![Add Tables to Your Diagrams](https://jcblogimages.blob.core.windows.net/img/2017/ssms-adding-tables.png)

Select the tables you want to learn more about and add them to your diagram. SSMS draws out any relationships that exist.

![A Simple Diagram in SSMS](https://jcblogimages.blob.core.windows.net/img/2017/ssms-simple-diagram.png)

By default, tables will be displayed with only the column names. A much more useful rendering is to switch to the "Standard" view which will reveal extended properties of the columns.

![Show Column Details in Your Database Diagram](https://jcblogimages.blob.core.windows.net/img/2017/ssms-show-table-details.png)

## Careful, There, Young Gunner

SSMS has a whole host of other features baked into diagrams that I tend to avoid. You can create new tables, modify columns, change properties and create relationships. 

**Don't.**

I can't see many scenarios where this would be a good idea. These types of things should be done through a managed and repeatable process that can be easily propogated to other developer workstations or out into production. Using the designer to do this will break my heart and also a kitten will die.

There is one case where this might be useful. If you are learning and want to better understand the SQL that is generated through the lens of something meaningful to you (like entities in your project) you might be able to use this as a learning tool. Go through with your changes in the designer (they are not saved by default), then select Database Diagram -> Generate Change Scripts from the application menu. SSMS will prompt you to save out change scripts.

**Note:** You may need to enable the setting called "Prevent saving changes that require table re-creation" in order to get some types of edits to work, otherwise you'll see a message saying that "Saving changes is not permitted."

![Changes to a Table](https://jcblogimages.blob.core.windows.net/img/2017/ssms-column-save.png)

You can enable that setting in Tools -> Options -> Designers. 

To it's credit, SSMS does a healthy job of trying to preserve your data. Here, I've simply added a column to the `CampaignGoal` table, but you can see what SSMS is trying to do behind the scenes with its change script:

```
/* To prevent any potential data loss issues, you should review this script in detail before running it outside the context of the database designer.*/
BEGIN TRANSACTION
SET QUOTED_IDENTIFIER ON
SET ARITHABORT ON
SET NUMERIC_ROUNDABORT OFF
SET CONCAT_NULL_YIELDS_NULL ON
SET ANSI_NULLS ON
SET ANSI_PADDING ON
SET ANSI_WARNINGS ON
COMMIT
BEGIN TRANSACTION
GO
ALTER TABLE dbo.CampaignGoal
	DROP CONSTRAINT FK_CampaignImpact_Campaign_CampaignId
GO
ALTER TABLE dbo.Campaign SET (LOCK_ESCALATION = TABLE)
GO
COMMIT
BEGIN TRANSACTION
GO
ALTER TABLE dbo.CampaignGoal
	DROP CONSTRAINT DF__CampaignI__Campa__0F624AF8
GO
CREATE TABLE dbo.Tmp_CampaignGoal
	(
	Id int NOT NULL IDENTITY (1, 1),
	CurrentGoalLevel int NOT NULL,
	Display bit NOT NULL,
	GoalType int NOT NULL,
	NumericGoal int NOT NULL,
	TextualGoal nvarchar(MAX) NULL,
	Notes nchar(10) NULL,
	CampaignId int NOT NULL
	)  ON [PRIMARY]
	 TEXTIMAGE_ON [PRIMARY]
GO
ALTER TABLE dbo.Tmp_CampaignGoal SET (LOCK_ESCALATION = TABLE)
GO
ALTER TABLE dbo.Tmp_CampaignGoal ADD CONSTRAINT
	DF__CampaignI__Campa__0F624AF8 DEFAULT ((0)) FOR CampaignId
GO
SET IDENTITY_INSERT dbo.Tmp_CampaignGoal ON
GO
IF EXISTS(SELECT * FROM dbo.CampaignGoal)
	 EXEC('INSERT INTO dbo.Tmp_CampaignGoal (Id, CurrentGoalLevel, Display, GoalType, NumericGoal, TextualGoal, CampaignId)
		SELECT Id, CurrentGoalLevel, Display, GoalType, NumericGoal, TextualGoal, CampaignId FROM dbo.CampaignGoal WITH (HOLDLOCK TABLOCKX)')
GO
SET IDENTITY_INSERT dbo.Tmp_CampaignGoal OFF
GO
DROP TABLE dbo.CampaignGoal
GO
EXECUTE sp_rename N'dbo.Tmp_CampaignGoal', N'CampaignGoal', 'OBJECT' 
GO
ALTER TABLE dbo.CampaignGoal ADD CONSTRAINT
	PK_CampaignImpact PRIMARY KEY CLUSTERED 
	(
	Id
	) WITH( STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]

GO
CREATE NONCLUSTERED INDEX IX_CampaignImpact_CampaignId ON dbo.CampaignGoal
	(
	CampaignId
	) WITH( STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
GO
ALTER TABLE dbo.CampaignGoal ADD CONSTRAINT
	FK_CampaignImpact_Campaign_CampaignId FOREIGN KEY
	(
	CampaignId
	) REFERENCES dbo.Campaign
	(
	Id
	) ON UPDATE  NO ACTION 
	 ON DELETE  CASCADE 
	
GO
COMMIT

```

Personally, I find `dotnet ef migrations add new-notes-field` a lot easier to do.

## Next Steps

Database Diagrams are local to your computer and are not part of your code base. If you're using EF and code-first, chances are you've never seen a "picture" of your model. Grab yourself a copy of [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms) and get sketching. 

That feature ain't going to finish itself ;)

Happy coding!