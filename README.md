# PIPE-Sqlite

This is a simple example of interfacing CMS Pipelines with the SQLITE database engine, using Pipe's co-routine capabilities.

The contents of the VMARC file are: 

1. SQL      FILE   _the input file of SQl statements_ 
2. SQLTEST  MODULE  
3. SQLTEST  LISTING 
4. SQLTEST  TEXT    
5. SQLTEST  C       _the C source file_

You will also need to install the [SQLITE](http://www.vm.ibm.com/download/packages/descript.cgi?SQLITE) package from the IBM VM Download page.

## The C souce code:
```C
#include <stdio.h>
#include <stdlib.h>
#include "sqlite3.h"
#include "fplpop.h"
   fplpopwa wa={0};
   fplpopwa ra={0};
   enum fplpopflag flag;
   char * wpipe="fitting FPLPOPEN|cons";
   char * rpipe="< sql file a | specs 1-* 1 x00 next | fitting FPLPOPEN";
   char buffer[200];
   char * line;
   int dlen;
   int rv;

static int callback(void *NotUsed, int argc, char **argv, char **azColName) {
   int i;

   for(i = 0; i<argc; i++) {
      sprintf(buffer, "%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
      dlen=strlen(buffer);
      rv=fplpwrite(&wa, buffer, &dlen);
      if (rv) printf("Return value %d on write 1.\n", rv);
     }
     return 0;
}

int main(int argc, char* argv[]) {
   sqlite3 *db;
   char *zErrMsg = 0;
   int rc;
   char *sql;
   const char* data = "Callback function called";
   const char* database = "test.db";

   /*  Open pipeline for write  */
   flag=fplpop_write;
   dlen=strlen(wpipe);
   rv=fplpopen(&wa, &flag, wpipe, &dlen);
   if (rv)
     {
     printf("Return value %d on open for write.\n", rv);
           return rv;
     }

   /*  Open pipeline for read   */
   flag=fplpop_read;
   dlen=strlen(rpipe);
   rv=fplpopen(&ra, &flag, rpipe, &dlen);
   if (rv)
     {
     printf("Return value %d on open for read .\n", rv);
           return rv;
     }

   /* Open database */
      rc = sqlite3_open(database, &db);
/*   rc = sqlite3_open(":memory:", &db); */

   if( rc ) {
      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
      fprintf(stderr, "RC =: %d\n", rc);
      return(0);
   } else {
      fprintf(stdout, "Opened database successfully\n");
   }

  fprintf(stdout, "database: %d %d \n", db, &db);
  fprintf(stdout, "callback: %d \n", callback);
   do
   {
   rv=fplpread(&ra,          &line, &dlen);
         if (rv)
           {
             printf("Return value %d on read.", rv);
             if (4==rv) printf("  That means EOF.\n");
             else printf("  That is an error.  Tell John.\n");
           }
           sql = line;
           /* Execute SQL statement */
           rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);

           if( rc != SQLITE_OK ){
              fprintf(stdout, "cmd: %d %s failed \n", rc, sql);
           fprintf(stdout, "SQL error: rc: %d, %s\n", rc, zErrMsg);
   fprintf(stdout, "%d, %s\n",sqlite3_extended_errcode(db),sqlite3_errmsg(db));
              sqlite3_free(zErrMsg);
           } else {
              fprintf(stdout, "cmd: %d %s done \n", rc, sql);
           }
           }
   while (0==rv);

   sqlite3_close(db);
   fplpclose(&wa);
   fplpclose(&ra);
   return 0;
   ```
## To compile and bind/link
### Compile
cc sqltest  c a (longname search(y) float(hex) LANGLVL(EXTENDED,LIBEXT,LONGLONG) source agg
### Bind
cmod sqltest  sqlite3 cmsvfs fplpop
    
## A sample run
```
sqltest
Opened database successfully
database: 524451144 524317368 
callback: 521045592 
cmd: 0 DROP TABLE IF EXISTS COMPANY; done 
cmd: 0 create table employee(empid integer,name varchar(20),title varchar(10)); done 
cmd: 0 create table department(deptid integer,name varchar(20),location varchar(10)); done 
cmd: 0 insert into employee values(101,'John Smith','CEO'); done 
cmd: 0 insert into employee values(102,'Raj Reddy','Sysadmin'); done 
cmd: 0 insert into employee values(103,'Jason Bourne','Developer'); done 
cmd: 0 insert into employee values(104,'Jane Smith','Sale Manager'); done 
cmd: 0 insert into employee values(105,'Rita Patel','DBA'); done 
cmd: 0   done 
cmd: 0 insert into department values(1,'Sales','Los Angeles'); done 
cmd: 0 insert into department values(2,'Technology','San Jose'); done 
cmd: 0 insert into department values(3,'Marketing','Los Angeles'); done 
empid = 101

name = John Smith

title = CEO

empid = 102

name = Raj Reddy

title = Sysadmin

empid = 103

name = Jason Bourne

title = Developer

empid = 104

name = Jane Smith

title = Sale Manager

empid = 105

name = Rita Patel

title = DBA

cmd: 0 select * from employee; done 
deptid = 1

name = Sales
   rc = sqlite3_open(database, &db);
/*   rc = sqlite3_open(":memory:", &db); */

   if( rc ) {
      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
      fprintf(stderr, "RC =: %d\n", rc);
      return(0);
   } else {
      fprintf(stdout, "Opened database successfully\n");
   }

  fprintf(stdout, "database: %d %d \n", db, &db);
  fprintf(stdout, "callback: %d \n", callback);
   do
   {
   rv=fplpread(&ra,          &line, &dlen);
         if (rv)
           {
             printf("Return value %d on read.", rv);
             if (4==rv) printf("  That means EOF.\n");
             else printf("  That is an error.  Tell John.\n");
           }
           sql = line;
           /* Execute SQL statement */
           rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);

           if( rc != SQLITE_OK ){
              fprintf(stdout, "cmd: %d %s failed \n", rc, sql);
           fprintf(stdout, "SQL error: rc: %d, %s\n", rc, zErrMsg);
   fprintf(stdout, "%d, %s\n",sqlite3_extended_errcode(db),sqlite3_errmsg(db));
              sqlite3_free(zErrMsg);
           } else {
              fprintf(stdout, "cmd: %d %s done \n", rc, sql);
           }
           }
   while (0==rv);

   sqlite3_close(db);
   fplpclose(&wa);
   fplpclose(&ra);
   return 0;
location = Los Angeles

deptid = 2

name = Technology

location = San Jose

deptid = 3

name = Marketing

location = Los Angeles

cmd: 0 select * from department; done 
cmd: 1 alter table department rename to dept; failed 
SQL error: rc: 1, ESCAPE expression must be a single character
1, ESCAPE expression must be a single character
cmd: 0 alter table employee add column deptid integer; done 
cmd: 0 update employee set deptid=3 where empid=101; done 
cmd: 0 update employee set deptid=2 where empid=102; done 
cmd: 0 update employee set deptid=2 where empid=103; done 
cmd: 0 update employee set deptid=1 where empid=104; done 
cmd: 0 update employee set deptid=2 where empid=105; done 
cmd: 0 create unique index empidx on employee(empid); done 
cmd: 0 alter table employee add column updatedon date; done 
cmd: 1 create trigger employee_update_trg after update on employee; failed 
SQL error: rc: 1, near ";": syntax error
1, near ";": syntax error
cmd: 0 begin done 
cmd: 1   update employee set updatedon = datetime('NOW') where rowid = new.rowid; failed 
SQL error: rc: 1, no such column: new.rowid
1, no such column: new.rowid
cmd: 0 end; done 
cmd: 0 update employee set title='Sales Manager' where empid=104; done 
empid = 101

name = John Smith

title = CEO

deptid = 3

updatedon = NULL

empid = 102

name = Raj Reddy

title = Sysadmin

deptid = 2

updatedon = NULL

empid = 103

name = Jason Bourne

title = Developer

deptid = 2

updatedon = NULL

empid = 104

name = Jane Smith

title = Sales Manager

deptid = 1

updatedon = NULL

empid = 105

name = Rita Patel

title = DBA

deptid = 2

updatedon = NULL

cmd: 0 select * from employee; done 
cmd: 0 create view empdept as select empid, e.name, title, d.name, location from employee e, dept d where e.deptid = d.deptid; done 
cmd: 1 select * from empdept; failed 
SQL error: rc: 1, no such table: main.dept
1, no such table: main.dept
Return value 4 on read.  That means EOF.
cmd: 0  done 
Ready; T=0.39/0.57 11:59:38
```

Don't forget to erase the generated "test.db" database file in the root BFS directory if you want to rerun the example and get the same results.
Good luck!
