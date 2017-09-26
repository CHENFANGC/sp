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



