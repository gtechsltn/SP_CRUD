# SP CRUD (SQL Server + ADO.NET)
+ Create table
+ Auto increment primary key column
+ Get auto increment primary key column after call SP Create/Insert
+ Stored Procedure CRUD
+ Adding a column description / Setting SQL Server Field Descriptions / Edit column descriptions
+ Set default value
+ Helper classes to call SPs from C# using ADO.NET

## TODO
+ Open SSMS
+ Copy & paste
+ Rename string "**YourTable**"
+ Replace some keywords by Sql Server Single Line Comment:
    + '#' -> '-- #'
    + 'SP' -> '-- SP'
+ Run script

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
    PRINT 'ALTER TABLE [dbo].[YourTable] ADD CONSTRAINT [DF_TaskStatus] DEFAULT 0 FOR [TaskStatus]: DONE'
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

# ADO.NET
## CRUD: C (Create)
```
public Int64 Create(YourTable yourTable)
{
	Int64 ID = default(Int64);
	try
	{
		using (var cnn = new SqlConnection(_connString))
		{
			cnn.Open();

			using (var cmd = cnn.CreateCommand())
			{
				cmd.CommandText = "YourTable_Insert";
				cmd.CommandType = CommandType.StoredProcedure;

				cmd.Parameters.Add(new SqlParameter("@Name", yourTable.Name.OrDBNull()));

				var objParamID = new SqlParameter("@ID", SqlDbType.BigInt);
				objParamID.Direction = ParameterDirection.Output;
				cmd.Parameters.Add(objParamID);

				var rowsAffected = cmd.ExecuteNonQuery();

				Int64? keyValue = objParamID.Value as Int64?;
				if (keyValue.HasValue)
				{
					ID = keyValue.Value;
					yourTable.ID = ID;
				}
			}

			cnn.Close();
		}

		return ID;
	}
	catch (Exception ex)
	{
		_log.Error(ex);
		throw;
	}
}
```

# Helper classes
```
using System;
using System.Data.SqlClient;

public static class DBWriteHelper
{
	public static object OrDBNull(this object someVar)
	{
		return someVar ?? DBNull.Value;
	}
}

public static class DBReadHelper
{
	public static T SafeGet<T>(this SqlDataReader reader, string colName)
	{
		var colIndex = reader.GetOrdinal(colName);
		return reader.IsDBNull(colIndex) ? default(T) : reader.GetFieldValue<T>(colIndex);
	}
}

public static class YourTableExtensions
{
	public static YourTable ToYourTable(this SqlDataReader rdr)
	{
		return new YourTable()
		{
			ID = SafeGet<Int64>(rdr, nameof(YourTable.ID)),
			Name = SafeGet<String>(rdr, nameof(YourTable.Name)),
			Created = SafeGet<DateTime>(rdr, nameof(YourTable.Created)),
			CreatedBy = SafeGet<Guid>(rdr, nameof(YourTable.CreatedBy)),
			Updated = SafeGet<DateTime>(rdr, nameof(YourTable.Updated)),
			UpdatedBy = SafeGet<Guid>(rdr, nameof(YourTable.UpdatedBy)),
			Deleted = SafeGet<DateTime?>(rdr, nameof(YourTable.Deleted)),
			DeletedBy = SafeGet<Guid?>(rdr, nameof(YourTable.DeletedBy)),
		};
	}

	public static YourTableDto ToYourTableDto(this SqlDataReader rdr)
	{
		return new YourTableDto()
		{
			ID = SafeGet<Int64>(rdr, nameof(YourTable.ID)),
			Name = SafeGet<String>(rdr, nameof(YourTable.AFName)),
		};
	}
}
```
