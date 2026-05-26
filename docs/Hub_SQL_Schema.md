**Airlink Hub — Chat History**

SQL Schema & Stored Procedures

DB: AirlinkDR  |  Version: 1.0  |  2/4/2026

# **1. New Tables**

## **1.1  Hub_ChatSessions**

One record per chat conversation. Tracks title, agent, timestamps, and soft-delete flag.

| **Column** | **Type** | **Default** | **Notes** |
| --- | --- | --- | --- |
| ChatID | uniqueidentifier | NEWID() | PK — auto-generated GUID |
| EmployeeCode | varchar(20) | — | FK-style: Empleados.Codigo |
| Agent | varchar(20) | — | 'CLAUDE' │ 'DINO' |
| Title | nvarchar(200) | 'Nueva conversación' | Auto-updated from first user message |
| CreatedAt | datetime | GETDATE() | — |
| UpdatedAt | datetime | GETDATE() | Updated on every new message |
| IsDeleted | bit | 0 | Soft delete — 1 = hidden from list |

**CREATE TABLE script:**

USE AirlinkDR;

GO

CREATE TABLE Hub_ChatSessions (

    ChatID       uniqueidentifier NOT NULL

                 CONSTRAINT PK_Hub_ChatSessions PRIMARY KEY

                 CONSTRAINT DF_HCS_ChatID DEFAULT NEWID(),

    EmployeeCode varchar(20)      NOT NULL,

    Agent        varchar(20)      NOT NULL,

    Title        nvarchar(200)    NOT NULL

                 CONSTRAINT DF_HCS_Title DEFAULT 'Nueva conversación',

    CreatedAt    datetime         NOT NULL

                 CONSTRAINT DF_HCS_CreatedAt DEFAULT GETDATE(),

    UpdatedAt    datetime         NOT NULL

                 CONSTRAINT DF_HCS_UpdatedAt DEFAULT GETDATE(),

    IsDeleted    bit              NOT NULL

                 CONSTRAINT DF_HCS_IsDeleted DEFAULT 0

);

CREATE INDEX IX_HCS_Employee_Agent

    ON Hub_ChatSessions (EmployeeCode, Agent, IsDeleted, UpdatedAt DESC);

GO

## **1.2  Hub_ChatMessages**

Stores every message within a chat session in order.

| **Column** | **Type** | **Default** | **Notes** |
| --- | --- | --- | --- |
| MsgID | bigint IDENTITY | auto | PK |
| ChatID | uniqueidentifier | — | FK → Hub_ChatSessions.ChatID |
| Seq | int | — | Message order (1-based) |
| Role | varchar(20) | — | 'user' │ 'assistant' |
| Content | nvarchar(MAX) | — | Full message text |
| CreatedAt | datetime | GETDATE() | — |

**CREATE TABLE script:**

CREATE TABLE Hub_ChatMessages (

    MsgID     bigint           NOT NULL

              CONSTRAINT PK_Hub_ChatMessages PRIMARY KEY IDENTITY(1,1),

    ChatID    uniqueidentifier NOT NULL

              CONSTRAINT FK_HCM_ChatID

              REFERENCES Hub_ChatSessions(ChatID) ON DELETE CASCADE,

    Seq       int              NOT NULL,

    Role      varchar(20)      NOT NULL,

    Content   nvarchar(MAX)    NOT NULL,

    CreatedAt datetime         NOT NULL

              CONSTRAINT DF_HCM_CreatedAt DEFAULT GETDATE()

);

CREATE INDEX IX_HCM_ChatID_Seq

    ON Hub_ChatMessages (ChatID, Seq ASC);

GO

-- Permissions

GRANT SELECT, INSERT, UPDATE, DELETE ON Hub_ChatSessions TO alappuser;

GRANT SELECT, INSERT, DELETE ON Hub_ChatMessages TO alappuser;

GO

# **2. Stored Procedures**

## **SP_Hub_UpsertChat**

Creates a new chat session or updates the title + timestamp of an existing one. Returns the ChatID.

CREATE OR ALTER PROCEDURE SP_Hub_UpsertChat

    @ChatID      uniqueidentifier,

    @EmployeeCode varchar(20),

    @Agent       varchar(20),

    @Title       nvarchar(200) = 'Nueva conversación'

AS

BEGIN

    SET NOCOUNT ON;

    IF EXISTS (SELECT 1 FROM Hub_ChatSessions WHERE ChatID = @ChatID)

    BEGIN

        -- Update title only if it was still the default

        UPDATE Hub_ChatSessions

        SET    Title     = CASE WHEN Title = 'Nueva conversación' THEN @Title ELSE Title END,

               UpdatedAt = GETDATE()

        WHERE  ChatID = @ChatID AND EmployeeCode = @EmployeeCode;

    END

    ELSE

    BEGIN

        INSERT INTO Hub_ChatSessions (ChatID, EmployeeCode, Agent, Title)

        VALUES (@ChatID, @EmployeeCode, @Agent, @Title);

    END

    SELECT ChatID FROM Hub_ChatSessions WHERE ChatID = @ChatID;

END

GO

GRANT EXECUTE ON SP_Hub_UpsertChat TO alappuser;

GO

## **SP_Hub_GetChatList**

Returns up to 50 active chats for an employee + agent, sorted newest first.

CREATE OR ALTER PROCEDURE SP_Hub_GetChatList

    @EmployeeCode varchar(20),

    @Agent        varchar(20),

    @Limit        int = 50

AS

BEGIN

    SET NOCOUNT ON;

    SELECT TOP (@Limit)

           ChatID,

           Title,

           CreatedAt,

           UpdatedAt

    FROM   Hub_ChatSessions

    WHERE  EmployeeCode = @EmployeeCode

      AND  Agent        = @Agent

      AND  IsDeleted    = 0

    ORDER  BY UpdatedAt DESC;

END

GO

GRANT EXECUTE ON SP_Hub_GetChatList TO alappuser;

GO

## **SP_Hub_SaveMessage**

Inserts a message and updates the session timestamp. Handles batch inserts from the Hub.

CREATE OR ALTER PROCEDURE SP_Hub_SaveMessage

    @ChatID  uniqueidentifier,

    @Seq     int,

    @Role    varchar(20),

    @Content nvarchar(MAX)

AS

BEGIN

    SET NOCOUNT ON;

    -- Insert message (ignore if duplicate Seq for idempotency)

    IF NOT EXISTS (SELECT 1 FROM Hub_ChatMessages

                   WHERE ChatID = @ChatID AND Seq = @Seq)

    BEGIN

        INSERT INTO Hub_ChatMessages (ChatID, Seq, Role, Content)

        VALUES (@ChatID, @Seq, @Role, @Content);

    END

    -- Touch session timestamp

    UPDATE Hub_ChatSessions

    SET    UpdatedAt = GETDATE()

    WHERE  ChatID = @ChatID;

    SELECT @Seq AS Seq;

END

GO

GRANT EXECUTE ON SP_Hub_SaveMessage TO alappuser;

GO

## **SP_Hub_GetChatMessages**

Returns all messages for a chat in order. EmployeeCode validated to prevent cross-user access.

CREATE OR ALTER PROCEDURE SP_Hub_GetChatMessages

    @ChatID      uniqueidentifier,

    @EmployeeCode varchar(20)

AS

BEGIN

    SET NOCOUNT ON;

    SELECT m.Seq, m.Role, m.Content, m.CreatedAt,

           s.Title, s.Agent

    FROM   Hub_ChatMessages  m

    JOIN   Hub_ChatSessions  s ON s.ChatID = m.ChatID

    WHERE  m.ChatID      = @ChatID

      AND  s.EmployeeCode = @EmployeeCode

      AND  s.IsDeleted    = 0

    ORDER  BY m.Seq ASC;

END

GO

GRANT EXECUTE ON SP_Hub_GetChatMessages TO alappuser;

GO

## **SP_Hub_DeleteChat**

Soft-deletes a chat (IsDeleted = 1). EmployeeCode validated to prevent cross-user deletion.

CREATE OR ALTER PROCEDURE SP_Hub_DeleteChat

    @ChatID      uniqueidentifier,

    @EmployeeCode varchar(20)

AS

BEGIN

    SET NOCOUNT ON;

    UPDATE Hub_ChatSessions

    SET    IsDeleted = 1,

           UpdatedAt = GETDATE()

    WHERE  ChatID       = @ChatID

      AND  EmployeeCode = @EmployeeCode;

    SELECT @@ROWCOUNT AS Deleted;

END

GO

GRANT EXECUTE ON SP_Hub_DeleteChat TO alappuser;

GO

# **3. Notes**

- Run CREATE TABLE scripts in AirlinkDR database, NOT SPN

- ON DELETE CASCADE on FK_HCM_ChatID: if a ChatSession is hard-deleted, messages cascade. Soft-delete via IsDeleted=1 is the normal path

- Seq column: Hub sends 1-based integer per message. SP_Hub_SaveMessage is idempotent — safe to retry

- Title logic: SP only updates title when it's still the default 'Nueva conversación' — first real user message wins

- alappuser must have SELECT, INSERT, UPDATE on both tables AND EXECUTE on all 5 SPs

- Index on (EmployeeCode, Agent, IsDeleted, UpdatedAt DESC) covers the GetChatList query fully