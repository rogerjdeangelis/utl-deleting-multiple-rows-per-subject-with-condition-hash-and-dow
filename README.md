# utl-deleting-multiple-rows-per-subject-with-condition-hash-and-dow
Delete emtire group if variable 1 eq 1 at least once  or variable 2 decreases.
    Deleting multiple rows per subject with condition hash and DOW

    This has multiple applications.

    Condition

      Delete emtire group if variable 1 eq 1 at least once  or variable 2 decreases;

    Two Solutions

      1. DOW loop
      2. Hash  (Novinosrin)

    github
    https://tinyurl.com/ya4wt7lk
    https://github.com/rogerjdeangelis/utl-deleting-multiple-rows-per-subject-with-condition-hash-and-dow

    see forum
    https://communities.sas.com/t5/New-SAS-User/deleting-multiple-rows-per-subject-with-condition/m-p/505305

    Novinosrin
    https://communities.sas.com/t5/user/viewprofilepage/user-id/138205


    INPUT
    =====
     WORK.HAVE total obs=9

      YEAR    ID    V1    V2    V3

        1      1     1    23     0
        2      1     2    23     0
        3      1     3    23     1
        1      2     1    14     0
        2      2     3    23     0
        3      2     2    23     0
        1      3     2    25     0
        2      3     2    12     0
        3      3     1    13     0

    RULES
    -----

     WORK.HAVE total obs=9

      YEAR    ID    V1    V2    V3

        1      1     1    23     0  Drop all records for id=1 because v3=1
        2      1     2    23     0
        3      1     3    23     1

        1      2     1    14     0
        2      2     3    23     0
        3      2     2    23     0

        1      3     2    25     0  Drop all records for id=3 because v2 decreases
        2      3     2    12     0
        3      3     1    13     0


    EXPECTED OUTPUT
    ---------------

     WORK.WANT total obs=3

      YEAR    ID    V1    V2    V3

        1      2     1    14     0
        2      2     3    23     0
        3      2     2    23     0

    PROCESS
    =======

    1. DOW loop

       data want;
        do until (last.id);
          set have;
          by id;
          if v3=1 or dif(v2)>0 then action='Delete Group';
          else action='Keep Group';
        end;
        put action=;
        do until (last.id);
          set have;
          by id;
          if action='Keep Group' then output;
        end;
        action='';
       run;quit;


    2. Hash  (Novinosrin)

       data _null_;
       if _n_=1 then do;
        dcl hash H (dataset:'have', multidata:'y') ;
          h.definekey  ("id") ;
          h.definedata (all:'y') ;
          h.definedone () ;
       end;
       set have end=lr;
       by id ;
       if first.id then call missing(v3_check);
       if v3=1 then v3_check=1;
       if not first.id and lag(v2)>v2 or last.id and v3_check=1 then rc=h.remove();
       if lr then h.output(dataset:'want');
       run;quit;

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    Data have;
     input year id v1 v2 v3;
    cards4;
    1 1 1 23 0
    2 1 2 23 0
    3 1 3 23 1
    1 2 1 14 0
    2 2 3 23 0
    3 2 2 23 0
    1 3 2 25 0
    2 3 2 12 0
    3 3 1 13 0
    ;;;;
    run;quit;

