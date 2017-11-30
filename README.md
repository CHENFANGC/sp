# sp
这也许是我目前写的最费劲的存储过程
USE [xxxxxxxxxxxx]
GO
/****** Object:  StoredProcedure [dbo].[P_Sys_GetRoleByRoleId]    Script Date: 2017/9/26 14:49:54 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<chenfang>
-- Create date: <2017/08/15>
-- Description:	<用户分配角色 角色查询功能>
-- =============================================
ALTER PROCEDURE [dbo].[P_Sys_GetRoleByRoleId]
	@PageIndex INT,--页码
	@PageSize INT,--每页多少条数据
	@Id  uniqueidentifier,--选择分配角色的用户Id
	--@Id  varchar(50),--选择分配角色的用户Id
	--@CheckBox bit,--是否分配 1:分配 0:未分配
	@Total int output--总条数
AS

BEGIN
	set @Total = (  SELECT count(1) 
					FROM SysRole
					WHERE SysRole.Active = 1 )

	--SELECT SysRole.Id , SysRole.Name, SysRole.[Description],
	--case when @Id in (select SysRoleSysUser.SysUserId from SysRoleSysUser) then 1 
    --        else 0
    --    end as CheckBox
	--FROM SysRole

	SELECT DISTINCT sr.Id ,
					sr.Name, 
					sr.[Description],
					CASE WHEN @Id IN (SELECT srsu.SysUserId FROM SysRoleSysUser srsu) AND 
						sr.Id IN (SELECT srsu.SysRoleId
									  FROM SysRoleSysUser srsu
									  WHERE srsu.SysUserId = @Id)
					THEN 1 ELSE 0
					END AS CheckBox
	FROM SysRole sr,SysRoleSysUser srsu
	WHERE sr.Active = 1
	ORDER BY sr.Name
	OFFSET (@PageIndex -1) * @PageSize ROWS
	FETCH NEXT @PageSize ROWS ONLY;
END


用户表：
USE [xxxxxxxxxxxx]
GO

/****** Object:  Table [dbo].[SysUser]    Script Date: 2017/9/26 14:54:23 ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[SysUser](
	[Id] [uniqueidentifier] NOT NULL,
	[DepId] [uniqueidentifier] NOT NULL,
	[PosId] [uniqueidentifier] NULL,
	[UserNO] [nvarchar](200) NOT NULL,
	[TrueName] [nvarchar](200) NULL,
	[Card] [nvarchar](50) NULL,
	[Sex] [nvarchar](10) NULL,
	[Birthday] [datetime] NULL,
	[Native] [nvarchar](200) NULL,
	[Higer] [nvarchar](200) NULL,
	[MobileNumber] [nvarchar](200) NULL,
	[PhoneNumber] [nvarchar](200) NULL,
	[QQ] [nvarchar](50) NULL,
	[EmailAddress] [nvarchar](200) NULL,
	[OtherContact] [nvarchar](200) NULL,
	[Province] [nvarchar](200) NULL,
	[City] [nvarchar](200) NULL,
	[Village] [nvarchar](200) NULL,
	[JoinDate] [datetime] NULL,
	[Marital] [nvarchar](10) NULL,
	[Political] [nvarchar](50) NULL,
	[Nationality] [nvarchar](20) NULL,
	[School] [nvarchar](50) NULL,
	[Expertise] [nvarchar](3000) NULL,
	[JobState] [bit] NOT NULL CONSTRAINT [DF__SysUser__JobStat__056ECC6A]  DEFAULT ((1)),
	[Professional] [nvarchar](100) NULL,
	[Degree] [nvarchar](20) NULL,
	[Photo] [nvarchar](200) NULL,
	[Attach] [nvarchar](500) NULL,
	[Lead] [nvarchar](4000) NULL,
	[LeadName] [nvarchar](4000) NULL,
	[IsSelLead] [bit] NOT NULL CONSTRAINT [DF__SysUser__IsSelLe__0662F0A3]  DEFAULT ((0)),
	[IsReportCalendar] [bit] NOT NULL CONSTRAINT [DF__SysUser__IsRepor__075714DC]  DEFAULT ((0)),
	[IsSecretary] [bit] NOT NULL CONSTRAINT [DF__SysUser__IsSecre__084B3915]  DEFAULT ((0)),
	[UserName] [nvarchar](200) NULL,
	[Password] [nvarchar](200) NULL,
	[Level] [nvarchar](10) NULL,
	[State] [smallint] NOT NULL CONSTRAINT [DF__SysUser__State__093F5D4E]  DEFAULT ((1)),
	[CreatedDate] [datetime] NOT NULL CONSTRAINT [DF__SysUser__Created__0A338187]  DEFAULT (getdate()),
	[CreatedUserID] [uniqueidentifier] NULL,
	[UpdatedDate] [datetime] NULL CONSTRAINT [DF_SysUser_UpdatedDate]  DEFAULT (getdate()),
	[UpdatedUserID] [uniqueidentifier] NULL,
	[WeChat] [nvarchar](50) NULL,
	[CloudHome] [nvarchar](50) NULL,
	[PosName] [nvarchar](200) NULL,
	[Synchronize] [smallint] NULL,
	[ServiceNetworkID] [uniqueidentifier] NULL,
 CONSTRAINT [PK_SysUser] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'微信' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'SysUser', @level2type=N'COLUMN',@level2name=N'WeChat'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'云之家' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'SysUser', @level2type=N'COLUMN',@level2name=N'CloudHome'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'0新增 1已编辑 2已同步' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'SysUser', @level2type=N'COLUMN',@level2name=N'Synchronize'
GO



角色表：
USE [xxxxxxxxxxxx]
GO

/****** Object:  Table [dbo].[SysRole]    Script Date: 2017/9/26 14:55:25 ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[SysRole](
	[Id] [uniqueidentifier] NOT NULL,
	[Name] [nvarchar](200) NOT NULL,
	[Description] [nvarchar](400) NULL,
	[Active] [smallint] NULL CONSTRAINT [DF_SysRole_Active]  DEFAULT ((1)),
	[CreatedDate] [datetime] NOT NULL CONSTRAINT [DF__SysRole__Created__00AA174D]  DEFAULT (getdate()),
	[CreatedUserID] [uniqueidentifier] NULL,
	[UpdatedDate] [datetime] NULL,
	[UpdatedUserID] [uniqueidentifier] NULL,
 CONSTRAINT [PK_SysRole] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO

用户与角色关联表：
USE [xxxxxxxxxxxx]
GO

/****** Object:  Table [dbo].[SysRoleSysUser]    Script Date: 2017/9/26 14:56:17 ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[SysRoleSysUser](
	[SysUserId] [uniqueidentifier] NOT NULL,
	[SysRoleId] [uniqueidentifier] NOT NULL,
	[CreatedDate] [datetime] NULL DEFAULT (getdate()),
	[CreatedUserID] [uniqueidentifier] NULL,
	[UpdatedDate] [datetime] NULL,
	[UpdatedUserID] [uniqueidentifier] NULL,
 CONSTRAINT [PK_SysRoleSysUser] PRIMARY KEY CLUSTERED 
(
	[SysUserId] ASC,
	[SysRoleId] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'用户ID' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'SysRoleSysUser', @level2type=N'COLUMN',@level2name=N'SysUserId'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'角色ID' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'SysRoleSysUser', @level2type=N'COLUMN',@level2name=N'SysRoleId'
GO

需求：
1.选择一个用户时，点击分配角色按钮，弹出添加角色画面；//程序实现，非sp
2.需要从用户与角色关联表中查找出是否有分配的角色，并将以分配的角色置为勾选状态；//存储过程实现
3.将新分配的角色在数据库中用户与角色关联表再存储，已有角色不重复添加。//程序实现，非sp

===================================================可爱的分割线(●'◡'●)==================================================================
关于sp中循环
写了以下sp才发现，没有最费劲，只有更费劲( ╯□╰ )————来自一个小白程序媛的内心
USE [GXCloudPlatform]
GO
/****** Object:  StoredProcedure [dbo].[P_MIS_GetTroubleCountByDeptForAnalayse]    Script Date: 2017/11/30 17:42:41 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


-- =============================================
-- Author:		<ChenFang>
-- Create date: <2017/11/07>
-- Description:	<故障按照部门柱状图画面用>
-- =============================================
ALTER PROCEDURE  [dbo].[P_MIS_GetTroubleCountByDeptForAnalayse]
	@DepId uniqueidentifier   --部门ID
AS
BEGIN

DECLARE @Name nvarchar(200) --部门名称
DECLARE @PeopleCount INT --根据传入的DepId在Struct表中查找的条数
DECLARE @TroubleCount INT --根据查找出的人员ID检索故障数
DECLARE @i int --计数
DECLARE @Total int --部门故障总数
DECLARE @Id uniqueidentifier --人员Id
DECLARE @devicetemp table (  Id uniqueidentifier ); --存储Ids的临时表
DECLARE @devicetemp1 table (  IndexNum int,Id uniqueidentifier ); --存储给@devicetemp加自增列的临时表
DECLARE @devicetemp2 table (  TroubleCount int ); 
INSERT INTO @devicetemp SELECT Id FROM [dbo].[SysUser] WHERE DepId = @DepId
INSERT INTO @devicetemp1 SELECT row_number() OVER(order by @@spid),* FROM @devicetemp

--Table1表中条数
SET @PeopleCount = (SELECT COUNT(Id) PeopleCount FROM @devicetemp)
SET @i = 1 
SET @Total = 0 
SET @Name =(
		SELECT ''+ Name 
		FROM  [dbo].[SysStruct]
		WHERE Id = @DepId FOR XML PATH('')
	)

--定义一个游标 
--declare rownum cursor for select @Ids FROM @devicetemp1

--打开游标 
--open rownum
--fetch next from rownum into @Ids
WHILE @i <= @PeopleCount  
	BEGIN
		SET @Id = (SELECT Id FROM @devicetemp1 where IndexNum = @i)	
		----根据查找出的人员ID检索故障数
		insert into @devicetemp2 SELECT COUNT(DISTINCT BusinessID) TroubleCount
								 FROM [dbo].[MIS_ComponentMaintenanceRecord]
								 WHERE MaintenanceUserID = @Id AND BusinessType = 1
		SET @TroubleCount = (SELECT SUM(TroubleCount) FROM @devicetemp2)
		SET @i = @i+1 
		--fetch next from rownum into @Ids
	END
	SET @Total = @Total + @TroubleCount
	SELECT @Name Name,@Total TroubleCount
	--PRINT @Total
--close rownum 
--摧毁游标 
--deallocate rownum
END


用户表：
USE [xxxxxxxxxxxx]
GO

/****** Object:  Table [dbo].[SysUser]    Script Date: 2017/9/26 14:54:23 ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[SysUser](
	[Id] [uniqueidentifier] NOT NULL,
	[DepId] [uniqueidentifier] NOT NULL,
	[PosId] [uniqueidentifier] NULL,
	[UserNO] [nvarchar](200) NOT NULL,
	[TrueName] [nvarchar](200) NULL,
	[Card] [nvarchar](50) NULL,
	[Sex] [nvarchar](10) NULL,
	[Birthday] [datetime] NULL,
	[Native] [nvarchar](200) NULL,
	[Higer] [nvarchar](200) NULL,
	[MobileNumber] [nvarchar](200) NULL,
	[PhoneNumber] [nvarchar](200) NULL,
	[QQ] [nvarchar](50) NULL,
	[EmailAddress] [nvarchar](200) NULL,
	[OtherContact] [nvarchar](200) NULL,
	[Province] [nvarchar](200) NULL,
	[City] [nvarchar](200) NULL,
	[Village] [nvarchar](200) NULL,
	[JoinDate] [datetime] NULL,
	[Marital] [nvarchar](10) NULL,
	[Political] [nvarchar](50) NULL,
	[Nationality] [nvarchar](20) NULL,
	[School] [nvarchar](50) NULL,
	[Expertise] [nvarchar](3000) NULL,
	[JobState] [bit] NOT NULL CONSTRAINT [DF__SysUser__JobStat__056ECC6A]  DEFAULT ((1)),
	[Professional] [nvarchar](100) NULL,
	[Degree] [nvarchar](20) NULL,
	[Photo] [nvarchar](200) NULL,
	[Attach] [nvarchar](500) NULL,
	[Lead] [nvarchar](4000) NULL,
	[LeadName] [nvarchar](4000) NULL,
	[IsSelLead] [bit] NOT NULL CONSTRAINT [DF__SysUser__IsSelLe__0662F0A3]  DEFAULT ((0)),
	[IsReportCalendar] [bit] NOT NULL CONSTRAINT [DF__SysUser__IsRepor__075714DC]  DEFAULT ((0)),
	[IsSecretary] [bit] NOT NULL CONSTRAINT [DF__SysUser__IsSecre__084B3915]  DEFAULT ((0)),
	[UserName] [nvarchar](200) NULL,
	[Password] [nvarchar](200) NULL,
	[Level] [nvarchar](10) NULL,
	[State] [smallint] NOT NULL CONSTRAINT [DF__SysUser__State__093F5D4E]  DEFAULT ((1)),
	[CreatedDate] [datetime] NOT NULL CONSTRAINT [DF__SysUser__Created__0A338187]  DEFAULT (getdate()),
	[CreatedUserID] [uniqueidentifier] NULL,
	[UpdatedDate] [datetime] NULL CONSTRAINT [DF_SysUser_UpdatedDate]  DEFAULT (getdate()),
	[UpdatedUserID] [uniqueidentifier] NULL,
	[WeChat] [nvarchar](50) NULL,
	[CloudHome] [nvarchar](50) NULL,
	[PosName] [nvarchar](200) NULL,
	[Synchronize] [smallint] NULL,
	[ServiceNetworkID] [uniqueidentifier] NULL,
 CONSTRAINT [PK_SysUser] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'微信' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'SysUser', @level2type=N'COLUMN',@level2name=N'WeChat'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'云之家' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'SysUser', @level2type=N'COLUMN',@level2name=N'CloudHome'
GO

EXEC sys.sp_addextendedproperty @name=N'MS_Description', @value=N'0新增 1已编辑 2已同步' , @level0type=N'SCHEMA',@level0name=N'dbo', @level1type=N'TABLE',@level1name=N'SysUser', @level2type=N'COLUMN',@level2name=N'Synchronize'
GO


部门表：
USE [xxxxxxxxxxxx]
GO

/****** Object:  Table [dbo].[SysStruct]    Script Date: 2017/11/30 17:45:39 ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

SET ANSI_PADDING ON
GO

CREATE TABLE [dbo].[SysStruct](
	[Id] [uniqueidentifier] NOT NULL,
	[Name] [nvarchar](200) NOT NULL,
	[ParentId] [uniqueidentifier] NULL,
	[Sort] [int] NOT NULL,
	[LeaderName] [nvarchar](200) NULL,
	[LeaderID] [varchar](max) NULL,
	[Enable] [smallint] NULL,
	[Remark] [nvarchar](500) NULL,
	[CreatedDate] [datetime] NULL,
	[CreatedUserID] [uniqueidentifier] NULL,
	[UpdatedDate] [datetime] NULL,
	[UpdatedUserID] [uniqueidentifier] NULL,
	[SructNo] [nvarchar](200) NOT NULL,
	[Address] [nvarchar](max) NULL,
 CONSTRAINT [PK_SysStruct] PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY],
 CONSTRAINT [uniqueStructNo] UNIQUE NONCLUSTERED 
(
	[SructNo] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]

GO

SET ANSI_PADDING OFF
GO

ALTER TABLE [dbo].[SysStruct] ADD  CONSTRAINT [DF__SysStruct__Sort__02925FBF]  DEFAULT ((0)) FOR [Sort]
GO

ALTER TABLE [dbo].[SysStruct] ADD  CONSTRAINT [DF__SysStruct__Enabl__038683F8]  DEFAULT ((1)) FOR [Enable]
GO

ALTER TABLE [dbo].[SysStruct] ADD  CONSTRAINT [DF__SysStruct__Creat__047AA831]  DEFAULT (getdate()) FOR [CreatedDate]
GO

维修记录表：
USE [xxxxxxxxxxxx]
GO

/****** Object:  Table [dbo].[MIS_ComponentMaintenanceRecord]    Script Date: 2017/11/30 17:52:10 ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[MIS_ComponentMaintenanceRecord](
	[RecordID] [uniqueidentifier] NOT NULL,
	[BusinessID] [uniqueidentifier] NULL,
	[BusinessType] [smallint] NOT NULL,
	[MaintenanceType] [smallint] NOT NULL,
	[MaintenanceUserID] [uniqueidentifier] NULL,
	[TroubleDealProcessDesc] [nvarchar](max) NULL,
	[IsNCR] [bit] NULL,
	[ImproveReportNo] [nvarchar](100) NULL,
	[CreatedDate] [datetime] NULL,
	[CreatedUserID] [uniqueidentifier] NULL,
	[UpdatedDate] [datetime] NULL,
	[UpdatedUserID] [uniqueidentifier] NULL,
	[MaintenanceManType] [int] NULL,
	[ComponentID] [bigint] NULL,
	[ComponentBOMID] [uniqueidentifier] NULL,
	[ProjectID] [uniqueidentifier] NULL,
	[ComponentTemplateID] [uniqueidentifier] NULL,
	[ComponentNO] [nvarchar](200) NULL,
	[ComponentName] [nvarchar](500) NULL,
	[ComponentType] [nvarchar](100) NULL,
	[ComponentXH] [nvarchar](100) NULL,
	[ProductUserState] [nvarchar](100) NULL,
	[BigMType] [nvarchar](100) NULL,
	[RunnedMileage] [decimal](18, 2) NULL,
	[CarNO] [nvarchar](200) NULL,
	[CarBoxNO] [nvarchar](200) NULL,
	[PeiShu] [nvarchar](200) NULL,
	[GuaranteePeriod] [datetime] NULL,
	[Productor] [nvarchar](500) NULL,
	[RTUFlagID] [nvarchar](100) NULL,
	[Encrypt] [nvarchar](50) NULL,
	[EncryptKey] [nvarchar](48) NULL,
	[ServerIP] [nvarchar](50) NULL,
	[IsRegister] [bit] NULL,
	[ServerPort] [nvarchar](10) NULL,
	[CommonRate] [smallint] NULL,
	[Active] [smallint] NULL,
	[Version] [int] NULL,
	[GPSXPos] [nvarchar](100) NULL,
	[GPSYPos] [nvarchar](100) NULL,
	[LastLoginDate] [datetime] NULL,
	[ComponentBOMNO] [nvarchar](200) NULL,
	[ComponentBOMProductor] [nvarchar](500) NULL,
	[ComponentBOMXH] [nvarchar](100) NULL,
	[IsProduct] [bit] NULL,
	[WorkState] [int] NULL,
	[PictureNO] [nvarchar](500) NULL,
	[MatterNO] [nvarchar](500) NULL,
	[ComponentGuaranteeMonths] [smallint] NULL,
	[NewComponentID] [bigint] NULL,
PRIMARY KEY CLUSTERED 
(
	[RecordID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]

GO

ALTER TABLE [dbo].[MIS_ComponentMaintenanceRecord] ADD  DEFAULT (newid()) FOR [RecordID]
GO

ALTER TABLE [dbo].[MIS_ComponentMaintenanceRecord] ADD  DEFAULT ((1)) FOR [BusinessType]
GO

ALTER TABLE [dbo].[MIS_ComponentMaintenanceRecord] ADD  DEFAULT ((0)) FOR [MaintenanceType]
GO

ALTER TABLE [dbo].[MIS_ComponentMaintenanceRecord] ADD  DEFAULT ((0)) FOR [IsNCR]
GO

ALTER TABLE [dbo].[MIS_ComponentMaintenanceRecord] ADD  DEFAULT (getdate()) FOR [CreatedDate]
GO

ALTER TABLE [dbo].[MIS_ComponentMaintenanceRecord] ADD  DEFAULT (getdate()) FOR [UpdatedDate]
GO


需求：
统计维修人所在部门处置故障数量

