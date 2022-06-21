-- hardcoding the table course_catalogue
CREATE TABLE  course_catalog (
   course_id text PRIMARY KEY,
   ltps text ,
   list_prerequisite text[],
   c int
);

-- hardcoding the table students

CREATE TABLE  students (
   student_id text PRIMARY KEY,
   student_name text,
   Dept_Name text,
   start_year text,
   user_name text
);

--hardcoding the instructors table 

CREATE TABLE  instructors (
   instructor_id text PRIMARY KEY,
   department text,
   name text,
   user_name text,
   batch_advisor_year text
);

--hardcoding academic semester table 

CREATE TABLE  academic_semester (
   year text,
   semester text
);

-- hardcoding the slots table 

CREATE TABLE slots(
    slot_id text PRIMARY KEY
);

-- hardcoding dean tickets table 



CREATE TABLE dean_tickets(
    student_id text,
    course_id text,
    semester text,
    year text,
    status text

);

-- creating a procedure for admin to insert items in the course_catalogue table;

create or replace procedure insert_course_catalog(
   course_id text ,
   ltps text ,
   prerequsite text,
   credit text
)
language plpgsql
as $$
declare
-- variable declaration
    dyn_query text;
begin
-- stored procedure body
    dyn_query='INSERT INTO course_catalog VALUES ( '||quote_literal(course_id)||','||quote_literal(ltps)||','||quote_literal(prerequsite)||','||credit||')';
    execute dyn_query;
end; $$;


-- creating a procedure for the admin to add a slot

create or replace procedure insert_slot(
   slot_id text
)
language plpgsql
as $$
declare
-- variable declaration
    dyn_query text;
begin
-- stored procedure body
    dyn_query='INSERT INTO slots VALUES (' ||quote_literal(slot_id) ||')';
    execute dyn_query;
end; $$;



-- procedure which will make student profile (only to be used by admin)

create or replace procedure create_student(
	user_name text,
	user_password text,
    student_id text,
    student_dept text,
    student_name text,
    start_year text
    )
    language plpgsql
    as $$
    declare
	    dyn_query text;
    begin
    -- Here the new user student is created and currently it will have no permissions
	
        dyn_query='CREATE USER '||user_name||' WITH PASSWORD '||quote_literal(user_password);
	    execute dyn_query;	

    -- Now inserting the entry corresponding to this student in the students table

        dyn_query='INSERT INTO students VALUES ( '|| quote_literal(student_id) ||' , ' || quote_literal(student_name) || ' , ' || 
                    quote_literal(student_dept) ||' , ' || quote_literal(start_year) ||' , ' || quote_literal(user_name) ||' )';

        execute dyn_query;

    -- Now creating the transcript table for this student

        dyn_query=' CREATE TABLE transcript_' || student_id || ' ( course_id text primary key ,
                                                                    year text,
                                                                    semester text,
                                                                    credits int,
                                                                    grade int ) ';
    
        execute dyn_query;

    -- Granting access to the student for viewing its own transcript table

        dyn_query=' GRANT SELECT ON transcript_'||student_id||' TO '||user_name;

        execute dyn_query;

    -- now creating the student_tickets table
    
        dyn_query=' CREATE TABLE tickets_' || student_id || ' ( course_id text primary key ,
                                                                    semester text,
                                                                    year text,
                                                                    status text ) ';
    
        execute dyn_query;

    -- granting access to view and insert its own ticket table

        dyn_query=' GRANT SELECT , INSERT ON tickets_'||student_id||' TO '||user_name;

        execute dyn_query;

    -- create student_id_current_courses table

        dyn_query=' CREATE TABLE current_courses_' || student_id || ' ( course_id text primary key ,
                                                                        slot text ,
                                                                        credits int) ';
        
        execute dyn_query;
                                                                
    -- grant permission to students to view and insert current course table

        dyn_query='GRANT SELECT , INSERT ON current_courses_'||student_id||' TO '||user_name;

        execute dyn_query;

    -- giving the student the permission to view the course catalog
    
        dyn_query=' GRANT SELECT ON course_catalog TO '||user_name;
        execute dyn_query;
    
    -- giving the student the permission to view the slots table

        dyn_query=' GRANT SELECT ON slots TO '||user_name;
        execute dyn_query;

    -- granting permission to student for insert and select on the tickets table

        dyn_query='GRANT SELECT , INSERT ON tickets_'||student_id||' TO '||user_name;

        execute dyn_query;

    -- granting permission to view students table

        dyn_query='GRANT SELECT ON students TO '||user_name;

        execute dyn_query;
    
    -- granting permission to view instructors table

        dyn_query='GRANT SELECT ON instructors TO '||user_name;

        execute dyn_query;

end; $$;



-- creating a procedure which will make instructor profile (to be used by admin)

create or replace procedure create_instructor(
    user_name text,
	user_password text,
    instructor_id text,
    instructor_dept text,
    instructor_name text
    )
    language plpgsql
    as $$
    declare
    -- variable declaration
    dyn_query text;
    begin

    -- Now the teacher user is created and currently it will not have any permissions

    dyn_query='CREATE USER '||user_name||' WITH PASSWORD '||quote_literal(user_password);
	execute dyn_query;
    
    -- Now inserting the entry corresponding to this user in the instructors table 
    
    dyn_query='INSERT INTO instructors VALUES ( '|| quote_literal(instructor_id) || ' , '|| quote_literal(instructor_dept) 
                || ' , ' || quote_literal(instructor_name) || ' , '|| quote_literal(user_name) ||','|| quote_literal('0000') ||' )';
    
    execute dyn_query;
   

    -- giving the instructor the permission to view the course catalog

    dyn_query=' GRANT SELECT ON course_catalog TO '||user_name;
    execute dyn_query;

    -- giving the instructor the permission to view the slots table

    dyn_query=' GRANT SELECT ON slots TO '||user_name;
    execute dyn_query;

    -- creating the instructor table 
    -- student id , course id , curr sem , curr year , status

    dyn_query = 'CREATE TABLE tickets_'||instructor_id||' ( student_id text,
                                                            course_id text,
                                                            semester text,
                                                            year text,
                                                            status text )';
    
    execute dyn_query;

    -- granting permission on  the tickets_table

    dyn_query='GRANT SELECT , UPDATE ON tickets_'||instructor_id||' TO '||user_name;

    execute dyn_query;

    
    -- granting permission to view students table

    dyn_query='GRANT SELECT ON students TO '||user_name;

    execute dyn_query;
    
    -- granting permission to view instructors table

    dyn_query='GRANT SELECT ON instructors TO '||user_name;

    execute dyn_query;


end; $$;



-- creating a function which will be used to view the details of a particular course in the course_catalogue table

create or replace function view_course_catalog (
    course_id_search text
) 
returns table ( course_id text,
        ltps text,   
        list_prerequsite text[],
        c int ) 
language plpgsql
as $$
declare 
	dyn_query text;
begin
-- body
	dyn_query='select * from course_catalog where course_id = '||quote_literal(course_id_search);
	return query
		execute dyn_query;
end; $$ ;


-- a function which will just display all the courses being currently offered 

create or replace function view_course_offerings (
	current_year text,
    current_sem  text
) 
returns table ( course_id text,
        instructor_id text,
        time_slot text,
		cgpa decimal(5,2),
		batches_allowed text[][]
         ) 
language plpgsql
as $$
declare 
	dyn_query text;
begin
-- body
	dyn_query='select * from course_offering_'||current_year||'_'||current_sem;
	return query
		execute dyn_query;
end; $$ ;



-- a procedure through which instructor offers course for the current semester

create or replace procedure offer_course(
        time_slot text,
		cgpa text,
		batches_allowed text,
        course_id text,
        instructor_id text,
        current_year text,
        current_sem text
)
language plpgsql
as $$
declare
-- variable declaration
    dyn_query text;
begin
-- simply inserting the specified row in the table 
    dyn_query='INSERT INTO course_offering_'||current_year||'_'||current_sem||' VALUES ( '|| quote_literal(course_id)  ||' , ' || quote_literal(instructor_id) || ' , ' || 
                    quote_literal(time_slot)  ||' , ' || cgpa ||' , ' ||  quote_literal(batches_allowed)||' )';
    execute dyn_query;

end; $$;



-- procedure to create the course_id_registration_sem_year tables and also giving them appropriate permissions

create or replace procedure create_course_registration_tables(
    current_year text,
    current_sem text
)
language plpgsql
as $$
declare
-- variable declaration
    dyn_query1 text;
    dyn_query2 text;
    dyn_query text;
    iterator record;
    iterator2 record;
begin
    dyn_query1='select * from course_offering_'||current_year||'_'||current_sem;
    for iterator in execute dyn_query1
    LOOP

        -- creating the registration_course_id_sem_year table

        dyn_query=' CREATE TABLE registration_' || iterator.course_id || '_'||current_sem||'_'||current_year||' ( student_id text primary key ) ';
        execute dyn_query;
        
        -- creating a trigger which will fire when a student attempts to insert a row on this registration_ table for this course

        dyn_query='create trigger '||iterator.course_id||'_'||current_sem||'_'||current_year||'_trigger before insert on registration_'||iterator.course_id
                                    ||'_'||current_sem||'_'||current_year||' for each row execute procedure trigger_register_course( '||
                                    quote_literal(iterator.course_id)||','||quote_literal(current_year)||','||quote_literal(current_sem)||
                                    ')';

        execute dyn_query;

        --creating the grades table for this course

        dyn_query=' CREATE TABLE grades_' || iterator.course_id ||'_'||current_sem||'_'||current_year||' ( student_id text primary key ,
                                                                                                grade int) ';
        execute dyn_query;



        -- giving appropriate permission to the particular instructor for this course
        
        EXECUTE 'select * from instructors where instructor_id = '||quote_literal(iterator.instructor_id)
        INTO iterator2;
        
        dyn_query=' GRANT select ON registration_'||iterator.course_id||'_'||current_sem||'_'||current_year||' TO '||iterator2.user_name;
        execute dyn_query;

        dyn_query=' GRANT ALL ON grades_' || iterator.course_id ||'_'||current_sem||'_'||current_year||' TO '||iterator2.user_name;
        execute dyn_query;

        

        -- giving the students appropriate permissions

        dyn_query2='select * from students';

        for iterator2 in execute dyn_query2
        LOOP
            dyn_query=' GRANT SELECT,INSERT ON registration_'||iterator.course_id||'_'||current_sem||'_'||current_year||' TO '||iterator2.user_name;
            execute dyn_query;
        END LOOP;



    END LOOP;

end; $$;



-- procedure that admin will call to put all the grades in the student_transcript table from the grades table of courses

create or replace procedure transcript_updation(
    current_year text,
    current_sem text
)
language plpgsql
as $$
declare
    iterator record;
    iterator2 record;
    get_rec record;
    dyn_query1 text;
    dyn_query2 text;    
    dyn_query text;
begin
-- stored procedure body
    
    -- iterating through the current sems offerings to get the course_ids.
    dyn_query1='select * from course_offering_'||current_year||'_'||current_sem;
    for iterator in execute dyn_query1
    loop 

        -- iterating through the grades table of different courses to update the student transcripts

        dyn_query2='select * from grades_'||iterator.course_id||'_'||current_sem||'_'||current_year;
    
        for iterator2 in execute dyn_query2
        loop 

            -- getting the record of the course from course_catalog to get the course_id
            EXECUTE 'select * from course_catalog where course_id='||quote_literal(iterator.course_id)
            INTO get_rec;

            -- now finally inserting into the transcript table if the grade is not fail
            if(iterator2.grade>3) then
            
                dyn_query='insert into transcript_'||iterator2.student_id||' values ( '||quote_literal(iterator.course_id)||','
                            ||quote_literal(current_year)||','||quote_literal(current_sem)||','||to_char(get_rec.c,'9')||','||to_char(iterator2.grade,'9')||' )';
                
                execute dyn_query;
            end if;
        end loop;

    end loop;
end; $$;

-- procedure to give grade to a student, to be called by the instructor

create or replace procedure grade_submit(
    student_id text,
    grade text,
    course text,
    year text,
    sem text
)
language plpgsql
as $$
declare
    dyn_query text;
begin
    dyn_query='insert into grades_'||course||'_'||sem||'_'||year||' values ('||quote_literal(student_id)||','||grade||')';
    execute dyn_query;
end; $$;



-- procedure to create batch advisor for a particular course

create or replace procedure create_batch_advisor(
    advisor_year text,
    advisor_id text
)
language plpgsql
as $$
declare
-- variable declaration
    dyn_query text;
    get_rec record;
begin
-- stored procedure body
    
    -- updating the required row in the instructors table 

    dyn_query = 'UPDATE instructors 
                  SET batch_advisor_year='||quote_literal(advisor_year)||
                 'WHERE instructor_id='||quote_literal(advisor_id);
    execute dyn_query;

    -- granting access to this instructor for the advisor table 
    execute 'SELECT * FROM instructors where instructor_id='||quote_literal(advisor_id)
            INTO get_rec;

    dyn_query = 'GRANT UPDATE , SELECT ON batch_advisor_'||advisor_year||' to '||get_rec.user_name;
    execute dyn_query;

end; $$;

create or replace procedure register_for_course(
    year text,
    sem text,
    course_id text,
    student_id text
)
language plpgsql
as $$
declare
-- variable declaration
    dyn_query text;
begin
-- stored procedure body
    dyn_query='INSERT INTO registration_'||course_id||'_'||sem||'_'||year||' values ( '||quote_literal(student_id)||')';
    execute dyn_query;
end; $$;



create or replace procedure insert_academic_sem(
    year text,
    semester text
)
language plpgsql
as $$
declare
    quer text;
    dyn_query text;
    quer1 text;
    student_row_rec record;
    curr_courses_table_name text;
begin
--TO begin the semester dean will be calling this function
    quer='INSERT into academic_semester values('||quote_literal(year)||','||quote_literal(semester)||')';
    EXECUTE quer;

    FOR student_row_rec IN select* from Students
    LOOP
	curr_courses_table_name:='current_courses_'||student_row_rec.Student_ID;
	quer1:='DELETE FROM '||curr_courses_table_name;
		execute quer1;
    END LOOP;
end;$$;



--table on which our trigger will act
create or replace function course_offertable ()
returns trigger
language plpgsql
as $$
declare
    table_name text;
    ins_row record;
    std_row record;
    quer text;
    quer2 text;
    quer3 text;
    dyn_query text;
    

begin


 
    -- first creating a new batch advisor table if required
    if(new.semester='1') then
        
        dyn_query='CREATE TABLE batch_advisor_'||new.year||' ( student_id text,
                                                                course_id text,
                                                                semester text,
                                                                year text,
                                                                status text )';
        execute dyn_query;
    end if;




    table_name = 'course_offering_'||new.year||'_'||new.semester;

    quer='create table '||table_name||' (course_id text NOT NULL PRIMARY KEY,
    Instructor_id text NOT NULL ,
    Time_Slot text NOT NULL,
    CGPA decimal(5,2) NOT NULL,
    Batches_Allowed_List text[][] NOT NULL
    )';

    EXECUTE quer;
                                                            
    -- initialising trigger
    dyn_query='create trigger course_offering_trigger_'||new.semester||'_'||new.year|| 
            ' before insert on Course_Offering_'||new.year||'_'||new.semester||
            ' for each row execute procedure trigger_instructor_validity()';
    
    execute dyn_query;

    -- NOW granting the permission to all the teachers to view,edit,update,delete from the course_offering table



    FOR ins_row IN select * from instructors
    LOOP
        quer2='GRANT SELECT,UPDATE,INSERT,DELETE on '||table_name||' to ' ||ins_row.user_name;
        EXECUTE quer2;
    END LOOP;
    FOR std_row IN select * from students
    LOOP
        quer3='GRANT SELECT on '||table_name||' to ' ||std_row.user_name;
        EXECUTE quer3;
    END LOOP;
    return new;
end; $$;




--to start the course offering table or to initialise it we will be putting trigger on the academic sem table
create trigger initialise_course
after insert on academic_semester
for each row
execute procedure course_offertable();


--course registration 
--TABLE NAME CHECK 
create or replace function trigger_register_course(
)
returns trigger
language plpgsql
as $$
declare
    curr_courseid text:=TG_ARGV[0];
    curr_year text:=TG_ARGV[1];
    curr_sem text:=TG_ARGV[2];

    curr_studentid text:=new.Student_id;
    curr_std_department text;
    curr_credits int;
    curr_std_strtyr text;
    req_timeslot text;
    req_cgpa decimal;
    curr_cgpa decimal;
    possible1 int:=0;
    --as per requirement
    possible2 int:=1;
    possible3 int:=0;
    --as per requirement
    possible4 int:=1;
    possible5 int:=0;
    CourseRegistration_table_name text='Registration_'||curr_courseid||'_'||curr_sem||'_'||curr_year;
    Studentranscript_table_name text:='transcript_'||curr_studentid;
    Currentregistered_table_name text:='current_courses_'||curr_studentid;
    Courseoffering_table_name text:='Course_Offering_'||curr_year||'_'||curr_sem;
    Batchallowed_list text[][];
    Prerequisites_list text[];
    --to get the data out from the dynamic tables
    courseoff record;
    catalog record;
    std_it record;
    quer1 text;

    --for cgpa
    total_sum decimal:=0.0;
    total_credits decimal:=0.0;
    std_row record;
    quer2 text;

    --for prereq
    s int:=0;
    course_cat record;
    c_id text;
    quer3 text;

    --for department and year check
    yr text[][];
    n int:=0;
    x text[];
    y text;
    year text;
    dep text;

    --for slot
    currregistered_row record;
    quer4 text;

    --for credit rule
    current_sem int;
    current_year int;
    avg_prev_credit decimal=0;
    iterator record;
    dyn_query text;
    sum decimal = 0;
    temp_sem int;
    temp_year int;
    temp_ctr decimal=0;
    current_total_credits decimal=0;
    course_credits decimal=0;
    iterator2 record;
begin

--intialising the batch allowed 2_d array using a for loop as previously done

    quer1='select * from '||Courseoffering_table_name||' where course_id='||quote_literal(curr_courseid);
    FOR courseoff IN execute quer1
    LOOP
      Batchallowed_list:=courseoff.Batches_Allowed_List;
      req_timeslot:=courseoff.Time_slot;
      req_cgpa:=courseoff.CGPA;
    END LOOP;


    --intialising the prereq 1d array using a loop as previously done 
  
    FOR catalog IN select * from Course_Catalog where Course_id=curr_courseid
    LOOP
        Prerequisites_list:=catalog.List_Prerequisite;
        curr_credits:=catalog.C;
    END LOOP;

--intialising the student's detials from the student's table
    
    execute 'select * from students where student_id='||quote_literal(curr_studentid)
	INTO std_it;
    

    curr_std_strtyr = std_it.Start_Year;
        
    curr_std_department = std_it.Dept_Name;

--raise notice '%', curr_studentid;
--all the intitial information is available.

--writing all five checks and keeping the track in one function only.

-- checking whether the user is inserting a row only on his behalf(check 0):- 

    -- extracting the username of the student_id being inserted 
    EXECUTE 'select * from students where student_id='||quote_literal(new.student_id)
    INTO iterator;

    -- now checking that it is same as the person who is inserting it
    select current_user into iterator2;
        if iterator2.current_user <> iterator.user_name then
	    raise notice 'Malicious insertion.';
            return NULL;
        end if;


--checking for cgpa(check 1)
    quer2='select * from '||Studentranscript_table_name;
    FOR std_row IN  execute quer2
    LOOP
        total_sum:=total_sum+std_row.credits*std_row.Grade;
        total_credits:=total_credits+std_row.credits;
    END LOOP;
    if(total_credits=0) then
        curr_cgpa=0;
    else
        curr_cgpa:=total_sum/total_credits;
    end if;

    IF curr_cgpa>=req_cgpa THEN
        possible1:=1;
    END IF;

--checking for prequisite courses(check 2)

    quer3='select * from '||Studentranscript_table_name;
    FOR course_cat IN select* from Course_catalog where Course_id=curr_courseid
    LOOP
        FOREACH c_id IN ARRAY course_cat.List_Prerequisite
        LOOP
        s:=0;
            FOR std_row IN execute quer3
            LOOP
                if c_id=std_row.Course_id THEN
                    s:=1;
                END IF;
            END LOOP;
            IF s=0 THEN 
                possible2=0;
            END IF;
        END LOOP;
        -- handling the case for no prerequsites
        if(course_cat.List_Prerequisite[1]='NA') then
            possible2=1;
        end if;

    END LOOP;

    




--checking for year and department(check 3)
    yr:=Batchallowed_list;
    FOREACH x SLICE 1 IN ARRAY yr
    LOOP
        year:=x[1];
        dep:=x[2];
        IF year=curr_std_strtyr and dep=curr_std_department THEN
            possible3=1;
        END IF;
    END LOOP;

--checking for slot in the currently reigistered courses table(check 4)
    quer4='select * from '||Currentregistered_table_name;
    FOR currregistered_row IN execute quer4
    LOOP
        IF currregistered_row.slot=req_timeslot THEN
            possible4:=0;
        END IF;
    END LOOP;

--checking for the 1.25 rule(check 5)

-- converting text year and sem to ints
	current_sem=to_number(curr_sem,'9');
	current_year=to_number(curr_year,'9999');

	-- extracting the course credits from the course_catalog table 
	execute 'select * from course_catalog where course_id='||quote_literal(curr_courseid) INTO iterator;
	course_credits=course_credits+iterator.c;
	
	-- have to convert current_sem and current_year into integers
	
	if current_sem=1 then
		temp_sem=2;
		temp_year=current_year-1;
		dyn_query='SELECT * from transcript_'||new.student_id||' as trans where trans.year='||
				quote_literal(trim(to_char(temp_year,'9999')))||' and '||'trans.semester='||
				quote_literal(trim(to_char(temp_sem,'9')));
		
		-- iterating through transcript of that particular student to sum credits of  last sem

        raise notice '%',dyn_query;

		for iterator in execute dyn_query
		loop
			temp_ctr=1;
			sum=sum+iterator.credits;
		end loop; 
		
		temp_sem=1;
		
		dyn_query='SELECT * from transcript_'||new.student_id||' as trans where trans.year='||
				quote_literal(trim(to_char(temp_year,'9999')))||' and '||'trans.semester='||
				quote_literal(trim(to_char(temp_sem,'9')));
		
		-- iterating through transcript of that particular student to sum credits of 2nd last sem

		for iterator in execute dyn_query
		loop
			temp_ctr=2;
			sum=sum+iterator.credits;
		end loop;	
		


		-- iterating through the student current table to find total credits registered till now
		
		dyn_query='SELECT * from current_courses_'||new.student_id;
		for iterator in execute dyn_query
		loop 
			current_total_credits=current_total_credits+iterator.credits;
		end loop;

		current_total_credits=current_total_credits+course_credits;		

        raise notice '%',temp_year;
        raise notice '%',temp_ctr;
        
		if temp_ctr<>0 then
			-- calculating 1.25 * avg of credits of the prev 2 sems 
			avg_prev_credit=sum/temp_ctr;
			avg_prev_credit=avg_prev_credit*1.25;
		end if;

		
		if avg_prev_credit=0 then
			possible5=1;
		elseif avg_prev_credit>=current_total_credits then
			possible5=1;
		else
			possible5=0;
		end if;
		
        
        raise notice '%',current_total_credits;
        raise notice '%',avg_prev_credit;
	
	elsif current_sem=2 then
		temp_sem=1;
		temp_year=current_year;
		dyn_query='SELECT * from transcript_'||new.student_id||' as trans where trans.year='||
				quote_literal(trim(to_char(temp_year,'9999')))||' and '||'trans.semester='||
				quote_literal(trim(to_char(temp_sem,'9')));
		
		-- iterating through transcript of that particular student to sum credits of  last sem

		for iterator in execute dyn_query
		loop
			temp_ctr=1;
			sum=sum+iterator.credits;
		end loop; 
		
		temp_sem=2;
		temp_year=current_year-1;
		
		dyn_query='SELECT * from transcript_'||new.student_id||' as trans where trans.year='||
				quote_literal(trim(to_char(temp_year,'9999')))||' and '||'trans.semester='||
				quote_literal(trim(to_char(temp_sem,'9')));
		
		-- iterating through transcript of that particular student to sum credits of 2nd last sem

		for iterator in execute dyn_query
		loop
			temp_ctr=2;
			sum=sum+iterator.credits;
		end loop;	
		


		-- iterating through the student current table to find total credits registered till now
		
		dyn_query='SELECT * from current_courses_'||new.student_id;
		for iterator in execute dyn_query
		loop 
			current_total_credits=current_total_credits+iterator.credits;
		end loop;

		current_total_credits=current_total_credits+course_credits;		

		if temp_ctr<>0 then
			-- calculating 1.25 * avg of credits of the prev 2 sems 
			avg_prev_credit=sum/temp_ctr;
			avg_prev_credit=avg_prev_credit*1.25;
		end if;

		
		if avg_prev_credit=0 then
			possible5=1;
		elseif avg_prev_credit>=current_total_credits then
			possible5=1;
		else
			possible5=0;
		end if;

	end if;

    IF possible1=1 and possible2=1 and possible3=1 and possible4=1 and possible5=1 THEN
        --current registered courses tableinsert into
        dyn_query='INSERT INTO '||Currentregistered_table_name ||' VALUES('|| quote_literal(curr_courseid) ||',' || quote_literal(req_timeslot) || ','   || to_char(curr_credits,'9') ||' )';
        execute dyn_query;
	raise notice 'Registration Successfull.';
        return new;
    ELSE
        raise notice 'You cannot register for this course as you do not fullfill the requirements  % % % % %',possible1,possible2,possible3,possible4,possible5;
        return null;
    END IF;
end;$$;



--tickets table
--to let a student raise a ticket and insert into the table corresponding to him
create or replace procedure generate_student_ticket(
    student_id text,
    courseid text,
    curr_sem text,
    curr_year text
) 
language plpgsql
as $$
declare
    tickets_table_name text:='tickets_'||student_id;
    quer1 text;
    quer2 text;
begin
    quer1='INSERT into '||tickets_table_name||' values(' ||quote_literal(courseid)|| ',' ||quote_literal(curr_sem)|| ',' ||quote_literal(curr_year)|| ',' ||quote_literal('NO')|| ')';
    execute quer1;
end; $$;


--to let the dean call the function so that the table corresponding to each and every other table in inserted
create or replace procedure propogate_all_tickets(
    curr_sem text,
    curr_year text
) 
language plpgsql
as $$
declare
    tickets_table_name text;
    tickets_instable_name text;
    tickets_batable_name text;
    tickets_deantable_name text;
    curr_courseoff_table_name text;
    curr_std_srtyr text;
    curr_insid text;
    quer1 text;
    quer2 text;
    quer3 text;
    quer4 text;
    quer5 text;
    std_row record;
    tickets_row record;
    curr_Instructor_id text;
    instructor_row record;
begin
    --quer1='INSERT into '||tickets_table_name||' values(' ||quote_literal(courseid)|| ',' ||quote_literal(curr_sem)|| ',' ||quote_literal(curr_year_)|| ','NO')';
     --execute quer1;

    FOR std_row IN select* from Students
    LOOP
        tickets_table_name :='tickets_'||std_row.Student_ID;
        
        curr_std_srtyr=std_row.Start_Year;
        tickets_batable_name :='batch_advisor_'||curr_std_srtyr;
        curr_courseoff_table_name :='Course_Offering_'||curr_year||'_'||curr_sem;
        quer1='select * from ' ||tickets_table_name||' where Year='||quote_literal(curr_year)||' and semester='||quote_literal(curr_sem);
        raise notice '%' , quer1;
        FOR tickets_row IN execute quer1
        LOOP

            

            --I will have to go in the course_offering table to find out the respective intructor
            quer2='select * from ' ||curr_courseoff_table_name||' where Course_id='||quote_literal(tickets_row.Course_id);
            FOR instructor_row IN execute quer2
            LOOP
                curr_Instructor_id=instructor_row.Instructor_id;
            END LOOP;

            --we have got the name of the instructor
            
            tickets_instable_name:='tickets_'||curr_Instructor_id;
            quer3='INSERT into '||tickets_instable_name||' values(' ||quote_literal(std_row.Student_ID)|| ',' ||quote_literal(tickets_row.Course_id)|| ',' ||quote_literal(curr_sem)|| ',' ||quote_literal(curr_year)||','||quote_literal('NO')||')';
            execute quer3;
        
            -- i will insert in the batch_advisor's table
            quer4='INSERT into '||tickets_batable_name||' values(' ||quote_literal(std_row.Student_ID)|| ',' ||quote_literal(tickets_row.Course_id)|| ',' ||quote_literal(curr_sem)|| ',' ||quote_literal(curr_year)|| ','||quote_literal('NO')||')';
            execute quer4;

            quer5='INSERT into dean_tickets values(' ||quote_literal(std_row.Student_ID)|| ',' ||quote_literal(tickets_row.Course_id)|| ',' ||quote_literal(curr_sem)|| ',' ||quote_literal(curr_year)|| ','||quote_literal('NO')||')';
            execute quer5;
        END LOOP;
    END LOOP;
    
end; $$;

--to update the intructors ticket table
create or replace procedure update_instructor_ticket_status(
    curr_std text,
    curr_sem text,
    curr_year text,
    his_id text,
    decision text,
    curr_course text
)
language plpgsql
as $$
declare
    table_name text:='tickets_'||his_id;
    quer1 text;
begin
    quer1:='UPDATE '||table_name||' SET status='||quote_literal(decision)||' WHERE student_id='||quote_literal(curr_std)||' and Year='||quote_literal(curr_year)||' and semester='||quote_literal(curr_sem)||' and course_id='||quote_literal(curr_course);
        execute quer1;
end;$$;

-- to update the batch_advisor's table
create or replace procedure update_batch_advisor_status(
    curr_std text,
    curr_sem text,
    curr_year text,
    decision text,
    curr_course text
)
language plpgsql
as $$
declare
    table_name text;
    curr_std_srtyr text;
    quer1 text;
    std_row record;
begin
    FOR std_row IN select* from Students where Student_ID=curr_std
    LOOP
        curr_std_srtyr :=std_row.Start_Year;
    END LOOP;
    
    table_name:='batch_advisor_'||curr_std_srtyr;
    quer1:='UPDATE '||table_name||' SET status='||quote_literal(decision)||' WHERE student_id='||quote_literal(curr_std)||' and Year='||quote_literal(curr_year)||' and Semester='||quote_literal(curr_sem)||' and course_id='||quote_literal(curr_course);
    execute quer1;
end;$$;



-- to update the dean_tickets table
create or replace procedure update_dean_tickets_status(
    curr_std text,
    curr_sem text,
    curr_year text,
    decision text,
    curr_course text
)
language plpgsql
as $$
declare
    tickets_table_name text:='tickets_'||curr_std;
    registration_table_name text:='Registration_'||curr_course||'_'||curr_sem||'_'||curr_year;
    currentreg_table_name text:='current_courses_'||curr_std;
    Courseoffering_table_name text:='Course_Offering_'||curr_year||'_'||curr_sem;
    trigger_name text:=curr_course||'_'||curr_sem||'_'||curr_year||'_trigger';
    quer1 text;
    quer2 text;
    quer3 text;
    quer4 text;
    quer5 text;
    quer6 text;
    quer7 text;
    curr_slot text;
    curr_credits text;
    courseoff record;
    catalog record;
begin
    
    
    --points here i need to update the student's table's status as well work on inserting a row in the registration as well as the current_courses table
    IF decision='YES' THEN
        --updating the student's ticket table
        
        
        quer1:='UPDATE '||tickets_table_name||' SET status='||quote_literal(decision)||' WHERE Year='||quote_literal(curr_year)||' and Semester='||quote_literal(curr_sem)||' and course_id='||quote_literal(curr_course);
        
        

        execute quer1;
        
     
        quer2:='UPDATE dean_tickets SET status='||quote_literal(decision)||' WHERE student_id='||quote_literal(curr_std)||' and Year='||quote_literal(curr_year)||' and Semester='||quote_literal(curr_sem)||' and course_id='||quote_literal(curr_course);
       
        execute quer2;
        --updating the dean's ticket table
        raise notice 'ola';
        -- adding to the current_registered_course of the student
        quer7:='select * from '||Courseoffering_table_name||' where course_id='||quote_literal(curr_course);
        FOR courseoff IN execute quer7
        LOOP
            curr_slot:=courseoff.Time_slot;
        END LOOP;
    
        FOR catalog IN select* from Course_Catalog where Course_id=curr_course
        LOOP
            curr_credits:=to_char(catalog.C,'9');
        END LOOP;

        quer3:='INSERT INTO '||currentreg_table_name||' VALUES('|| quote_literal(curr_course) ||',' || quote_literal(curr_slot) || ','   ||curr_credits||' )';
        execute quer3;

        --inserting into 
        quer4:='ALTER TABLE '||registration_table_name||' DISABLE TRIGGER '||trigger_name;
        execute quer4;
        quer5:='INSERT INTO '||registration_table_name||' VALUES('|| quote_literal(curr_std) ||' )';
        execute quer5;
        quer6:='ALTER TABLE '||registration_table_name||' ENABLE TRIGGER '||trigger_name;
        execute quer6;
    ELSE
        quer1:='UPDATE '||tickets_table_name||' SET status='||quote_literal(decision)||' WHERE Year='||quote_literal(curr_year)||' and Semester='||quote_literal(curr_sem)||' and course_id='||quote_literal(curr_course);
            execute quer1;
        quer2:='UPDATE dean_tickets SET status='||quote_literal(decision)||' WHERE student_id='||quote_literal(curr_std)||' and Year='||quote_literal(curr_year)||' and Semester='||quote_literal(curr_sem)||' and course_id='||quote_literal(curr_course);
            execute quer2;
    END IF;
end;$$;

-- to calculate cgpa of student

create or replace procedure  calc_cgpa(
    student_id text
)
language plpgsql
as $$
declare
    total_sum decimal:=0.0;
    total_credits decimal:=0.0;
    transcript_row record;
    table_name text:='transcript_'||student_id;
    --table_name text:=student_id||'_transcript';
    quer text;
begin
    quer='select* from '||table_name;
    FOR transcript_row IN  execute quer
    LOOP
        total_sum:=total_sum+transcript_row.credits*transcript_row.Grade;
        total_credits:=total_credits+transcript_row.credits;
    END LOOP;
    if total_sum = 0 then 
        raise notice 'No courses yet.';
    else
        raise notice 'your current cgpa is %',total_sum/total_credits;
    end if;
end; $$;


-- trigger function for validating course_offering insertion

CREATE OR REPLACE FUNCTION trigger_instructor_validity() RETURNS TRIGGER AS $$
    declare
        iterator record;
        iterator2 record;
    BEGIN
       -- extracting the username of the instructor being inserted 
    EXECUTE 'select * from Instructors where Instructor_id='||quote_literal(new.Instructor_id)
    INTO iterator;

    -- now checking that it is same as the person who is inserting it
    select current_user into iterator2;
        if iterator2.current_user <> iterator.user_name then
	        raise notice 'Malicious insertion.';
            return NULL;
        end if;
        return new;
    END;
$$ LANGUAGE plpgsql;



