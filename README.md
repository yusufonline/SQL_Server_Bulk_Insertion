# SQL_Server_Bulk_Insertion
Secondary School App that register students with all their respective subjects from their catagory eg jss or ss
insert a record in table student_basic_data then run stored procedure for class allocation to allocate student to class called table result, you have supply the session eg (2022/2023)
In order to test the bulk insertion stored procedure theres is a table ClassbyClass you need to add the present class and a class you want to promote to,there by runing the stored procedure ClassbyClass_promo then you can supply the session year and term.
Below is the code sample 
USE [CloudSchoolCleanCopy]
GO
/****** Object:  StoredProcedure [dbo].[Stored_ClassByClass_Promo]    Script Date: 10/24/2022 5:19:56 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER  PROCEDURE [dbo].[Stored_ClassByClass_Promo]
(	
	--@PresentClass varchar(15),
    --@Promoclass varchar(15),
	@PresentCat  varchar(15),
	@PresentYear varchar(15),	
	@PresentTerm varchar(15),
	@PromoCat   varchar(15),
	@PromoYear  varchar(15),
	@PromoTerm  varchar(15),
	@OrgCode as int
)
AS   
BEGIN
	SET NOCOUNT ON;	
	DECLARE @myClass_table TABLE (ClassRowID INT IDENTITY(1,1),OrgCode int,PresentClass nvarchar(15),Promoclass nvarchar(15));
	DECLARE @myResults_table TABLE (FileRowID INT IDENTITY(1,1),OrgCode int,Cat nvarchar(15),Acad_Year nvarchar(15),Term nvarchar(15),Classid nvarchar(15),FileID nvarchar(15));
	DECLARE @mysubjects_table TABLE (subRowID INT IDENTITY(1,1),OrgCode int,Subj_Id nvarchar(15),Cat_Id  nvarchar(15));
	DECLARE @MySkills_table TABLE (SkillRowID INT IDENTITY(1,1),OrgCode int, SkillID nvarchar(15));
	DECLARE @MyBehavior_table TABLE (BehRowID INT IDENTITY(1,1),OrgCode int, BehID nvarchar(15));
	
	DECLARE @ResultsRow as int, @ResultsCount as int;
	DECLARE @SubJRow as int, @SubJCount as int;
	DECLARE @ClassRow as int, @ClassCount as int;
	DECLARE @SkillRow as Int, @SkillCount as int;
	DECLARE @BehRow as int, @BehCount as int,@mySkillID as varchar(50),@myBehID as varchar(50), @BehID as varchar(50)
    DECLARE @myFileID as varchar(15),@mysubj as varchar(15)
	DECLARE @PresentClass as varchar(15),@Promoclass as varchar(15);
	
	--@ClassCount as varchar(15)	 						
	--Get al the Students List from a particular class Set ready for Promotion!
	---------------------------------------------------------------------------------------------------- Start 1
	INSERT INTO @myClass_table(OrgCode,PresentClass,Promoclass)
	SELECT Org_Code, PClass,PromoClass From ClassbyClass

	SELECT @ClassCount= @@RowCount, @ClassRow  = 1 	

	WHILE @ClassRow <=@ClassCount
	BEGIN

	SELECT @PresentClass=PresentClass,@Promoclass=Promoclass FROM @myClass_table WHERE OrgCode=@OrgCode and  @ClassRow=ClassRowID 
	---------------------------------------------------------------------------------------------------- Start 2
		INSERT INTO @myResults_table(OrgCode,Cat,Acad_Year ,Term ,Classid,FileID) 
		SELECT DISTINCT Org_Code, Category, Academic_Year, Term, Class_ID, File_ID FROM Results WHERE
		(Org_Code=@OrgCode) and(Category = @PresentCat) AND (Academic_Year = @PresentYear) AND (Term = @Presentterm) AND (Class_ID = @PresentClass) 
		ORDER BY Org_Code, Category, Academic_Year, Term, Class_ID,File_ID																			
						
		SELECT @Resultscount = @@RowCount, @ResultsRow = 1 						
			
		WHILE @ResultsRow  <= @Resultscount		
		BEGIN
		--Specify the Student and Present Class			
		SELECT @myFileID=FileID FROM @myResults_table WHERE Orgcode=@OrgCode and FileRowID = @Resultsrow														
		
		--Specify the Promoted Class						
		---------------------------------------------------------------------------------------------------- Start 3
		INSERT INTO @mysubjects_table(OrgCode,Subj_Id,Cat_Id) 
		SELECT Org_Code,Subj_ID, Subcategory FROM  Subject_Type  WHERE (Org_Code=@OrgCode) and (Subcategory = @PromoCat)
							
		SELECT @SubJcount = @@RowCount, @SubJRow =1
							
		WHILE @SubJRow <=@SubJcount 
		BEGIN		
			  SELECT @mysubj=Subj_Id FROM @mysubjects_table WHERE OrgCode=@OrgCode and subRowID = @SubJRow
					--Results Tables
					INSERT INTO Results(Org_Code,Category, Academic_Year, Term, File_ID, Class_ID, Subj_ID)
					VALUES (@OrgCode,@PromoCat, @PromoYear, @PromoTerm, @myFileID, @PromoClass, @mysubj)								
				
				  SET @SubJRow = @SubJrow + 1  
		END							
		---------------------------------------------------------------------------------------------------- End 3
		-- Skills Tables
		INSERT INTO @MySkills_table(OrgCode,SkillID)
		SELECT Org_Code,Skills_ID FROM Skills where Org_Code=@OrgCode

		 SELECT @SkillCount=@@RowCount, @SkillRow =1

		 WHILE @SkillRow <= @SkillCount 
		 BEGIN
				SELECT @mySkillID=SkillID From @MySkills_table  WHERE (Orgcode=@OrgCode) and (SkillRowID=@SkillRow)
					insert into  StudentSkills(Org_Code,Category,Academic_year,Term,File_id,Class_id,SkillID)
					Values (@OrgCode,@PromoCat, @PromoYear, @PromoTerm, @myFileID, @PromoClass, @mySkillID)			
			SET @SkillRow = @SkillRow + 1
		 END 
		 	 
		---------------------------------------------------------------------------------------------------- Emd 
		--- Behaviors Tables	 
		 INSERT INTO @MyBehavior_table(OrgCode,BehID)
		 SELECT Org_Code, B_ID FROM Behaviour  where Org_Code=@OrgCode
		 
		  SELECT @Behcount=@@RowCount, @BehRow=1		  		  
		  WHILE @BehRow <= @Behcount
		  BEGIN
					SELECT @BehID=BehID FROM @MyBehavior_table WHERE (OrgCode=@OrgCode) and (BehRowID=@BehRow)
					insert into StudentBehaviour(Org_Code,Category,Academic_year,Term,File_id,Class_id,BehaviourID)
					Values (@OrgCode,@PromoCat, @PromoYear, @PromoTerm, @myFileID, @PromoClass, @BehID)	
			
			SET @BehRow = @BehRow + 1
		   END		 		
		---------------------------------------------------------------------------------------------------- ENd 
		SET @Resultsrow = @Resultsrow + 1  
		END
		----------------------------------------------------------------------------------------------------End 1
		SET @ClassRow=@ClassRow+1
	END 
END


