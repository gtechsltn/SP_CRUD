# SP CRUD
+ Create table
+ Auto increment primary key column
+ Get auto increment primary key column after call SP Create/Insert
+ Stored Procedure CRUD
+ Adding a column description / Setting SQL Server Field Descriptions / Edit column descriptions
+ Set default value

## TODO
+ Rename string "**YourTable**"
+ Replace some keywords by Sql Server Single Line Comment: '#' -> '-- #'

## DROP TABLE
```
DROP TABLE [dbo].[YourTable]
```

## CREATE TABLE IF NOT EXISTS
```
--======================================================================================================================================================
IF (NOT EXISTS(SELECT * FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'dbo' AND TABLE_NAME = 'YourTable'))
BEGIN
    CREATE TABLE [dbo].[YourTable] (
	[ID] [int] IDENTITY (1,1) NOT NULL
	,[Name] [varchar](500) NULL
	,[TaskStatus] [int]
	,[Created] [datetime] NOT NULL
	,[CreatedBy] [uniqueidentifier] NOT NULL
	,[Updated] [datetime] NOT NULL
	,[UpdatedBy] [uniqueidentifier] NOT NULL
	,[Deleted] [datetime] NULL
	,[DeletedBy] [uniqueidentifier] NULL
	,CONSTRAINT [PK_YourTable] PRIMARY KEY CLUSTERED 
	(
	    [ID] ASC
	)
    )
END
GO
```

# SP Default Value
```
-- ====================================================================================================
IF NOT EXISTS (SELECT * FROM sys.objects WHERE OBJECT_ID = OBJECT_ID(N'[DF_TaskStatus]') AND TYPE = 'D')
BEGIN
    --ALTER TABLE [dbo].[YourTable] ADD CONSTRAINT [DF_TaskStatus] DEFAULT 0 FOR [TaskStatus];
    PRINT 'ALTER TABLE [dbo].[YourTable] ADD CONSTRAINT [DF_TaskStatus] DEFAULT 0 FOR [TaskStatus];'
END
GO
```

## SP CRUD (C) ~ Create: YourTable_Insert
```
-- ====================================================================================================
IF EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'YourTable_Insert') AND type IN (N'P', N'PC'))
    DROP PROCEDURE [dbo].[YourTable_Insert]
GO

-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[YourTable_Insert] (
    @ID BIGINT OUT,
    @Name NVARCHAR(255)
)
AS
BEGIN
    INSERT INTO [dbo].[YourTable] (
        [Name]
    )
    VALUES (
        @Name
    )
    --Retrieve SQL Server identity column values
    SET @ID = SCOPE_IDENTITY()
END
GO
```
## SP CRUD (R) ~ Read: YourTable_GetByID
```
-- ====================================================================================================
IF EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'YourTable_GetByID') AND type IN (N'P', N'PC'))
    DROP PROCEDURE [dbo].[YourTable_GetByID]
GO

-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[YourTable_GetByID] (@ID BIGINT)
AS
BEGIN
	SELECT [ID]
		,[Name]
		,[Created]
		,[CreatedBy]
		,[Updated]
		,[UpdatedBy]
		,[Deleted]
		,[DeletedBy]
	FROM [dbo].[YourTable]
	WHERE [ID] = @ID
END
GO
```

## SP CRUD (U) ~ Update: YourTable_Update
```
-- ====================================================================================================
IF EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'YourTable_Update') AND type IN (N'P', N'PC'))
    DROP PROCEDURE [dbo].[YourTable_Update]
GO

-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[YourTable_Update] (
	@ID BIGINT
	,@Name NVARCHAR(255)
	,@Updated DATETIME
	,@UpdatedBy UNIQUEIDENTIFIER
	,@Deleted DATETIME
	,@DeletedBy UNIQUEIDENTIFIER
)
AS
BEGIN
	UPDATE [dbo].[YourTable]
	SET [Name] = @Name
		,[Updated] = @Updated
		,[UpdatedBy] = @UpdatedBy
		,[Deleted] = @Deleted
		,[DeletedBy] = @DeletedBy
	WHERE [ID] = @ID
END
GO
```

## SP CRUD (D) ~ Delete: YourTable_Delete
```
-- ====================================================================================================
IF EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'YourTable_Delete') AND type IN (N'P', N'PC'))
    DROP PROCEDURE [dbo].[YourTable_Delete]
GO

-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[YourTable_Delete] (
    @ID BIGINT
    ,@UserUid VARCHAR(36) = ''
)
AS
BEGIN
    UPDATE [dbo].[YourTable]
    SET [Deleted] = GETUTCDATE()
        ,[DeletedBy] = @UserUid
    WHERE [ID] = @ID
END
GO
```

# SP: common_SetDescription
```
-- ====================================================================================================
IF EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'common_SetDescription') AND type IN (N'P', N'PC'))
    DROP PROCEDURE [dbo].[common_SetDescription]
GO

-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
CREATE PROCEDURE [dbo].[common_SetDescription] (
    @tableName NVARCHAR(255)
    ,@columnName NVARCHAR(255) = NULL
    ,@objectDescription NVARCHAR(255)
)
AS
BEGIN
    IF (@columnName IS NULL)
    BEGIN
	IF NOT EXISTS (
			SELECT 1
			FROM fn_listextendedproperty(NULL, 'user', 'dbo', 'table', DEFAULT, NULL, NULL)
			WHERE OBJNAME = @tableName
			)
	BEGIN
		EXECUTE sp_addextendedproperty 'MY_DESCRIPTION'
			,@objectDescription
			,'user'
			,dbo
			,'table'
			,@tableName
			,DEFAULT
			,NULL
	END
	ELSE
	BEGIN
		EXECUTE sp_updateextendedproperty 'MY_DESCRIPTION'
			,@objectDescription
			,'user'
			,dbo
			,'table'
			,@tableName
			,DEFAULT
			,NULL
	END
    END
    ELSE
    BEGIN
	IF NOT EXISTS (
			SELECT 1
			FROM sys.extended_properties AS ep
			INNER JOIN sys.tables AS t
				ON ep.major_id = t.object_id
			INNER JOIN sys.columns AS c
				ON ep.major_id = c.object_id
					AND ep.minor_id = c.column_id
			WHERE class = 1
				AND t.name = @tableName
				AND c.name = @columnName
			)
	BEGIN
		EXECUTE sp_addextendedproperty 'MS_Description'
			,@objectDescription
			,'user'
			,dbo
			,'table'
			,@tableName
			,'column'
			,@columnName
	END
	ELSE
	BEGIN
		EXECUTE sp_updateextendedproperty 'MS_Description'
			,@objectDescription
			,'user'
			,dbo
			,'table'
			,@tableName
			,'column'
			,@columnName
	END
    END
END
GO

-- ====================================================================================================
EXEC [dbo].[common_SetDescription] @tableName = N'YourTable'
    ,@columnName = N'TaskStatus'
    ,@objectDescription = N'Status Code = 1: Scheduled, 2: Processing, 3: Completed, 4: Failed'
GO
```
