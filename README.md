# utl-matching-tables-based-on-a-hierachy-of-rules-sas-hash-sql-r-pytho-excel
Matching tables based on a hierarchy of rules sas hash sql r pytho excel
    %let pgm=utl-matching-tables-based-on-a-hierachy-of-rules-sas-hash-sql-r-pytho-excel;

    %stop_submission;

    Matching tables based on a hierarhy of rules sas hash sql r python excel

    PROBLEM (HIERRACHICAL JOIN AND UPDATE0

    MUST BE DONE IN THIS ORDER (THATS WHAT MAKES THIS HARD)

    if name age sex match then select val from master
    else if name and age match then select val from master
    else if name match then select val from master
    else keep remaining master records se val=999;

      CONTENTS

         1 sas hash (best solution)
         2 sas sql  (not sure this covers all cases? maybe id variables assures the order?)
         3 r sql
           see for python excel
           https://tinyurl.com/ptn3a9fx
         4 related hash repositories

    github
    https://tinyurl.com/y6p5v3aj
    https://github.com/rogerjdeangelis/utl-matching-tables-based-on-a-hierachy-of-rules-sas-hash-sql-r-pytho-excel

    for python excel
    github
    https://tinyurl.com/ptn3a9fx
    https://github.com/rogerjdeangelis/utl-simple-example-of-a-three-table-full-outer-join-by-ids-sas-r-python-excel-sql

    communities.sas
    https://tinyurl.com/ys8vwesj
    https://communities.sas.com/t5/SAS-Programming/I-need-to-match-data-from-2-separate-data-sets-based-on-multiple/m-p/823724#M325281

    /**************************************************************************************************************************/
    /*       INPUT           | MUST BE DONE IN THIS ORDER                          |  NAME AGE SEX VAL  MATCH_LEVEL           */
    /*       =====           |                                                     |                                          */
    /*                       | if name age and sex match then keep val from master | Alex   19  X   1 no match                */
    /*    SD1.MASTER         | else if name and age match then keep val from master| Alfred 14  M   2 matched name+age+sex    */
    /* ----------------      | else if name match then keep val from master        | Alice  13  F   4 matched name+age+sex    */
    /* ID NAME  AGE SEX VAL  | else keep remaining master records wgt=999;         | Alice  13  U   6 matched name+age        */
    /*                       |                                                     | Alice  19  U   8 matched name            */
    /* 1  Alfred  14  M  2   |                                                     | Marg   19  X   0 no match                */
    /* 2  Alice   13  F  4   |    SD1.MASTER            SD1.TRANS                  |                                          */
    /* 3  Alice   13  U  6   |    ----------------  ------------------------------ |                                          */
    /* 4  Alice   19  U  8   | ID NAME  AGE SEX VAL|  NAME  AGE SEX MATCHES        |                                          */
    /* 5  Marg    19  X  0   |                     |                               |                                          */
    /* 6  Alex    19  X  1   | 1  Alfred  14  M  2 | Alfred  14  M  name age sex   |                                          */
    /*                       | 2  Alice   13  F  4 | Alice   13  F  name age sex   |                                          */
    /*     SD1.TRANS         | 3  Alice   13  U  6 | Alice   13  Y  name age       |                                          */
    /*  --------------       | 4  Alice   19  U  8 | Alice   22  Z  name           |                                          */
    /*   NAME  AGE SEX       | 5  Marg    19  X  0 |                no match       |                                          */
    /*                       | 6  Alex    19  X  1 |                no match       |                                          */
    /*  Alfred  14  M        |                                                     |                                          */
    /*  Alice   13  F        | OUTPUT                                              |                                          */
    /*  Alice   13  Y        |   NAME  AGE SEX VAL MATCH_LEVEL                     |                                          */
    /*  Alice   22  Z        |                                                     |                                          */
    /*                       |  Alex    19  X   1  no match                        |                                          */
    /* options               |  Alfred  14  M   2  matched on name+age+sex         |                                          */
    /*  validvarname=upcase; |  Alice   13  F   4  matched on name+age+sex         |                                          */
    /* libname sd1 "d:/sd1"; |  Alice   13  U   6  matched on name+age             |                                          */
    /* data sd1.master;      |  Alice   19  U   8  matched on name                 |                                          */
    /*   input id name$      |  Marg    19  X   0  no match                        |                                          */
    /*     age sex$ val;     |                                                     |                                          */
    /* cards4;               |------------------------------------------------------------------------------------------------*/
    /* 1 Alfred 14 M 2       |                                                     |                                          */
    /* 2 Alice  13 F 4       |  1 SAS HASH (ELEGANT AND SCALABLE)                  | ID     NAME     AGE    SEX    VAL        */
    /* 3 Alice  13 U 6       |  ===================================                |                                          */
    /* 4 Alice  19 U 8       |                                                     |  1    Alfred     14     M       2        */
    /* 5 Marg   19 X 0       |  data want;                                         |  2    Alice      13     F       4        */
    /* 6 Alex   19 X 1       |                                                     |  3    Alice      13     U       6        */
    /* ;;;;                  |   if _n_=1 then                                     |  4    Alice      19     U       8        */
    /* run;quit;             |     do;                                             |  5    Marg       19     X     999        */
    /*                       |       if 0 then set sd1.trans;                      |  6    Alex       19     X     999        */
    /* data sd1.trans;       |                                                     |                                          */
    /*   input id name$      |       dcl hash h_rule1(dataset:'sd1.trans');        |                                          */
    /*     age sex$ ;        |       h_rule1.defineKey('name','age','sex');        |                                          */
    /* cards4;               |       h_rule1.defineDone();                         |                                          */
    /* 1 Alfred  14 M        |                                                     |                                          */
    /* 2 Alice   13 F        |       dcl hash h_rule2(dataset:'sd1.trans');        |                                          */
    /* 3 Alice   13 Y        |       h_rule2.defineKey('name','age');              |                                          */
    /* 4 Alice   22 Z        |       h_rule2.defineDone();                         |                                          */
    /* ;;;;                  |                                                     |                                          */
    /* run;quit;             |       dcl hash h_rule3(dataset:'sd1.trans');        |                                          */
    /*                       |       h_rule3.defineKey('name');                    |                                          */
    /*                       |       h_rule3.defineDone();                         |                                          */
    /*                       |                                                     |                                          */
    /*                       |     end;                                            |                                          */
    /*                       |                                                     |                                          */
    /*                       |   call missing(of _all_);                           |                                          */
    /*                       |                                                     |                                          */
    /*                       |   set sd1.master;                                   |                                          */
    /*                       |                                                     |                                          */
    /*                       |   if      h_rule1.find()=0 then;                    |                                          */
    /*                       |   else if h_rule2.find()=0 then;                    |                                          */
    /*                       |   else if h_rule3.find()=0 then;                    |                                          */
    /*                       |   else val=999;                                     |                                          */
    /*                       |                                                     |                                          */
    /*                       |  run;quit;                                          |                                          */
    /*                       |                                                     |                                          */
    /*                       |------------------------------------------------------------------------------------------------*/
    /*                       |                                                     |                                          */
    /*                       |  2 SAS SQL                                          | ID     NAME     AGE    SEX    VAL        */
    /*                       |  =========                                          |                                          */
    /*                       |                                                     |  1    Alfred     14     M       2        */
    /*                       |  proc sql;                                          |  2    Alice      13     F       4        */
    /*                       |  create                                             |  3    Alice      13     U       6        */
    /*                       |      table want as                                  |  4    Alice      19     U       8        */
    /*                       |  select                                             |  5    Marg       19     X     999        */
    /*                       |      distinct                                       |  6    Alex       19     X     999        */
    /*                       |      t1.name, t1.age, t1.sex, t1.val,               |                                          */
    /*                       |      case                                           |                                          */
    /*                       |          when t2a.id is not null                    |                                          */
    /*                       |            then 'matched on name+age+sex'           |                                          */
    /*                       |          when t2b.id is not null                    |                                          */
    /*                       |            then 'matched on name+age'               |                                          */
    /*                       |          when t2c.id is not null                    |                                          */
    /*                       |            then 'matched on name'                   |                                          */
    /*                       |          else 'no match'                            |                                          */
    /*                       |      end as match_level                             |                                          */
    /*                       |  from sd1.master t1                                 |                                          */
    /*                       |  left join sd1.trans t2a                            |                                          */
    /*                       |      on t1.name = t2a.name and                      |                                          */
    /*                       |         t1.age  = t2a.age   and                     |                                          */
    /*                       |         t1.sex  = t2a.sex                           |                                          */
    /*                       |  left join sd1.trans t2b                            |                                          */
    /*                       |      on t2a.id is null and                          |                                          */
    /*                       |         t1.name = t2b.name and                      |                                          */
    /*                       |         t1.age = t2b.age                            |                                          */
    /*                       |  left join sd1.trans t2c                            |                                          */
    /*                       |      on t2a.id is null and                          |                                          */
    /*                       |         t2b.id is null                              |                                          */
    /*                       |      and t1.name = t2c.name;                        |                                          */
    /*                       |  ;quit;                                             |                                          */
    /*                       |                                                     |                                          */
    /*                       |------------------------------------------------------------------------------------------------*/
    /*                       |                                                     |R                                         */
    /*                       |  3 R SQL                                            |    NAME AGE SEX VAL       match_level    */
    /*                       |  =======                                            |1 Alfred  14   M   2 matched name+age+sex */
    /*                       |                                                     |2  Alice  13   F   4 matched name+age+sex */
    /*                       |  %utl_rbeginx;                                      |3  Alice  13   U   6     matched name+age */
    /*                       |  parmcards4;                                        |4  Alice  19   U   8         matched name */
    /*                       |  library(haven)                                     |5   Marg  19   X   0             no match */
    /*                       |  library(sqldf)                                     |6   Alex  19   X   1             no match */
    /*                       |  source("c:/oto/fn_tosas9x.R")                      |                                          */
    /*                       |  master<-read_sas("d:/sd1/master.sas7bdat")         |                                          */
    /*                       |  trans<-read_sas("d:/sd1/trans.sas7bdat")           |SAS                                       */
    /*                       |  print(master)                                      | ID     NAME     AGE    SEX    VAL        */
    /*                       |  print(trans)                                       |                                          */
    /*                       |  want<-sqldf("                                      |  1    Alfred     14     M       2        */
    /*                       |    select                                           |  2    Alice      13     F       4        */
    /*                       |        distinct                                     |  3    Alice      13     U       6        */
    /*                       |        t1.name, t1.age, t1.sex, t1.val,             |  4    Alice      19     U       8        */
    /*                       |        case                                         |  5    Marg       19     X     999        */
    /*                       |            when t2a.id is not null                  |  6    Alex       19     X     999        */
    /*                       |              then 'matched on name+age+sex'         |                                          */
    /*                       |            when t2b.id is not null                  |                                          */
    /*                       |              then 'matched on name+age'             |                                          */
    /*                       |            when t2c.id is not null                  |                                          */
    /*                       |              then 'matched on name'                 |                                          */
    /*                       |            else 'no match'                          |                                          */
    /*                       |        end as match_level                           |                                          */
    /*                       |    from master t1                                   |                                          */
    /*                       |    left join trans t2a                              |                                          */
    /*                       |        on t1.name = t2a.name and                    |                                          */
    /*                       |           t1.age  = t2a.age   and                   |                                          */
    /*                       |           t1.sex  = t2a.sex                         |                                          */
    /*                       |    left join trans t2b                              |                                          */
    /*                       |        on t2a.id is null and                        |                                          */
    /*                       |           t1.name = t2b.name and                    |                                          */
    /*                       |           t1.age = t2b.age                          |                                          */
    /*                       |    left join trans t2c                              |                                          */
    /*                       |        on t2a.id is null and                        |                                          */
    /*                       |           t2b.id is null                            |                                          */
    /*                       |        and t1.name = t2c.name;                      |                                          */
    /*                       |                                                     |                                          */
    /*                       |    ")                                               |                                          */
    /*                       |  want                                               |                                          */
    /*                       |  fn_tosas9x(                                        |                                          */
    /*                       |        inp    = want                                |                                          */
    /*                       |       ,outlib ="d:/sd1/"                            |                                          */
    /*                       |       ,outdsn ="want"                               |                                          */
    /*                       |       )                                             |                                          */
    /*                       |  ;;;;                                               |                                          */
    /*                       |  %utl_rendx;                                        |                                          */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options
     validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.master;
      input id name$
        age sex$ val;
    cards4;
    1 Alfred 14 M 2
    2 Alice  13 F 4
    3 Alice  13 U 6
    4 Alice  19 U 8
    5 Marg   19 X 0
    6 Alex   19 X 1
    ;;;;
    run;quit;

    data sd1.trans;
      input id name$
        age sex$ ;
    cards4;
    1 Alfred  14 M
    2 Alice   13 F
    3 Alice   13 Y
    4 Alice   22 Z
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /* SD1.MASTER                                                                                                             */
    /*                                                                                                                        */
    /*  ID     NAME     AGE    SEX    VAL                                                                                     */
    /*                                                                                                                        */
    /*   1    Alfred     14     M      2                                                                                      */
    /*   2    Alice      13     F      4                                                                                      */
    /*   3    Alice      13     U      6                                                                                      */
    /*   4    Alice      19     U      8                                                                                      */
    /*   5    Marg       19     X      0                                                                                      */
    /*   6    Alex       19     X      1                                                                                      */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /*  SD1.TRANS                                                                                                             */
    /*  ID     NAME     AGE    SEX                                                                                            */
    /*                                                                                                                        */
    /*   1    Alfred     14     M                                                                                             */
    /*   2    Alice      13     F                                                                                             */
    /*   3    Alice      13     Y                                                                                             */
    /*   4    Alice      22     Z                                                                                             */
    /**************************************************************************************************************************/

    /*                   _               _
    / |  ___  __ _ ___  | |__   __ _ ___| |__
    | | / __|/ _` / __| | `_ \ / _` / __| `_ \
    | | \__ \ (_| \__ \ | | | | (_| \__ \ | | |
    |_| |___/\__,_|___/ |_| |_|\__,_|___/_| |_|

    */

    data want;

     if _n_=1 then
       do;
         if 0 then set sd1.trans;

         dcl hash h_rule1(dataset:'sd1.trans');
         h_rule1.defineKey('name','age','sex');
         h_rule1.defineDone();

         dcl hash h_rule2(dataset:'sd1.trans');
         h_rule2.defineKey('name','age');
         h_rule2.defineDone();

         dcl hash h_rule3(dataset:'sd1.trans');
         h_rule3.defineKey('name');
         h_rule3.defineDone();

       end;

     call missing(of _all_);

     set sd1.master;

     if      h_rule1.find()=0 then;
     else if h_rule2.find()=0 then;
     else if h_rule3.find()=0 then;
     else val=999;

    run;quit;

    /**************************************************************************************************************************/
    /*  ID     NAME     AGE    SEX    VAL                                                                                     */
    /*                                                                                                                        */
    /*   1    Alfred     14     M       2                                                                                     */
    /*   2    Alice      13     F       4                                                                                     */
    /*   3    Alice      13     U       6                                                                                     */
    /*   4    Alice      19     U       8                                                                                     */
    /*   5    Marg       19     X     999                                                                                     */
    /*   6    Alex       19     X     999                                                                                     */
    /**************************************************************************************************************************/

    /*___                              _
    |___ \   ___  __ _ ___   ___  __ _| |
      __) | / __|/ _` / __| / __|/ _` | |
     / __/  \__ \ (_| \__ \ \__ \ (_| | |
    |_____| |___/\__,_|___/ |___/\__, |_|
                                    |_|
    */

    proc sql;
    create
        table want as
    select
        distinct
        t1.name, t1.age, t1.sex, t1.val,
        case
            when t2a.id is not null
              then 'matched on name+age+sex'
            when t2b.id is not null
              then 'matched on name+age'
            when t2c.id is not null
              then 'matched on name'
            else 'no match'
        end as match_level
    from sd1.master t1
    left join sd1.trans t2a
        on t1.name = t2a.name and
           t1.age  = t2a.age   and
           t1.sex  = t2a.sex
    left join sd1.trans t2b
        on t2a.id is null and
           t1.name = t2b.name and
           t1.age = t2b.age
    left join sd1.trans t2c
        on t2a.id is null and
           t2b.id is null
        and t1.name = t2c.name;
    ;quit;

    /**************************************************************************************************************************/
    /* WANT                                                                                                                   */
    /*    NAME     AGE    SEX    VAL    MATCH_LEVEL                                                                           */
    /*                                                                                                                        */
    /*   Alex       19     X      1     no match                                                                              */
    /*   Alfred     14     M      2     matched on name+age+sex                                                               */
    /*   Alice      13     F      4     matched on name+age+sex                                                               */
    /*   Alice      13     U      6     matched on name+age                                                                   */
    /*   Alice      19     U      8     matched on name                                                                       */
    /*   Marg       19     X      0     no match                                                                              */
    /**************************************************************************************************************************/

    /*____                    _
    |___ /   _ __   ___  __ _| |
      |_ \  | `__| / __|/ _` | |
     ___) | | |    \__ \ (_| | |
    |____/  |_|    |___/\__, |_|
                           |_|
    */


    3 R SQL
    =======

    %utl_rbeginx;
    parmcards4;
    library(haven)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
    master<-read_sas("d:/sd1/master.sas7bdat")
    trans<-read_sas("d:/sd1/trans.sas7bdat")
    print(master)
    print(trans)
    want<-sqldf("
      select
          distinct
          t1.name, t1.age, t1.sex, t1.val,
          case
              when t2a.id is not null
                then 'matched on name+age+sex'
              when t2b.id is not null
                then 'matched on name+age'
              when t2c.id is not null
                then 'matched on name'
              else 'no match'
          end as match_level
      from master t1
      left join trans t2a
          on t1.name = t2a.name and
             t1.age  = t2a.age   and
             t1.sex  = t2a.sex
      left join trans t2b
          on t2a.id is null and
             t1.name = t2b.name and
             t1.age = t2b.age
      left join trans t2c
          on t2a.id is null and
             t2b.id is null
          and t1.name = t2c.name;

      ")
    want
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    /**************************************************************************************************************************/
    /* R                                           SAS                                                                        */
    /*     NAME AGE SEX VAL       match_level       ID     NAME     AGE    SEX    VAL                                         */
    /* 1 Alfred  14   M   2 matched name+age+sex     1    Alfred     14     M       2                                         */
    /* 2  Alice  13   F   4 matched name+age+sex     2    Alice      13     F       4                                         */
    /* 3  Alice  13   U   6     matched name+age     3    Alice      13     U       6                                         */
    /* 4  Alice  19   U   8         matched name     4    Alice      19     U       8                                         */
    /* 5   Marg  19   X   0             no match     5    Marg       19     X     999                                         */
    /* 6   Alex  19   X   1             no match     6    Alex       19     X     999                                         */
    /**************************************************************************************************************************/

    /*  _
    | || |    _ __ ___ _ __   ___  ___
    | || |_  | `__/ _ \ `_ \ / _ \/ __|
    |__   _| | | |  __/ |_) | (_) \__ \
       |_|   |_|  \___| .__/ \___/|___/
                      |_|
    */

    REPO
    ------------------------------------------------------------------------------------------------------------------------------------
    https://github.com/rogerjdeangelis/distinct-counts-for_3200-variables-and_660-thousand-records-using-HASH-SQL-and-proc-freq
    https://github.com/rogerjdeangelis/utl-append-and-split-tables-into-two-tables-one-with-common-variables-and-one-without-dosubl-hash
    https://github.com/rogerjdeangelis/utl-are-the-files-identical-or-was-the-file-corrupted-durring-transfer-hash
    https://github.com/rogerjdeangelis/utl-average-nap-time-for-three-babies-in-and-unsorted-table-using-a-hash-and-r
    https://github.com/rogerjdeangelis/utl-count-distinct-compound-keys-using-sql-and-hash-algorithms
    https://github.com/rogerjdeangelis/utl-create-a-list-of-male-students-at-achme-high-school-using_a_hash
    https://github.com/rogerjdeangelis/utl-create-a-state-diagram-table-hash-corresp-and-transpose
    https://github.com/rogerjdeangelis/utl-creating-two-tables-sum-of-weight-by-age-and-by-sex-using-a-hash-of-hashes_hoh
    https://github.com/rogerjdeangelis/utl-deduping-six-hundred-million-records-with-one-million-unique-sql-hash
    https://github.com/rogerjdeangelis/utl-deleting-multiple-rows-per-subject-with-condition-hash-and-dow
    https://github.com/rogerjdeangelis/utl-dosubl-persistent-hash-across-datasteps-and-procedures
    https://github.com/rogerjdeangelis/utl-elegant-hash-to-add-missing-weeks-by-customer
    https://github.com/rogerjdeangelis/utl-excluding-patients-that-had-same-condition-pre-and-post-clinical-randomization-hash
    https://github.com/rogerjdeangelis/utl-fast-efficient-hash-to-eliminate-duplicates-in-unsorted-grouped-data
    https://github.com/rogerjdeangelis/utl-fast-join-small_1g-table_with-a-moderate_50gb-tables-hash-sql
    https://github.com/rogerjdeangelis/utl-fast-normalization-and-join-using-vvaluex-arrays-sql-hash-untranspose-macro
    https://github.com/rogerjdeangelis/utl-hash-applying-business-rules-by-observation-when-data-and-rules-are-in-the-same-table
    https://github.com/rogerjdeangelis/utl-hash-filling-in-missing-gender-for-my-patients-appointments
    https://github.com/rogerjdeangelis/utl-hash-of-hashes-left-join-four-tables
    https://github.com/rogerjdeangelis/utl-hash-vs-summary-min-and-max-for-four-variables-by-region-for-l89-million-obs
    https://github.com/rogerjdeangelis/utl-hash_which-columns-have-duplicate-values-across-rows
    https://github.com/rogerjdeangelis/utl-in-memory-hash-output-shared-with-dosubl-hash-subprocess
    https://github.com/rogerjdeangelis/utl-loop-through-one-table-and-find-data-in-next-table--hash-dosubl-arts-transpose
    https://github.com/rogerjdeangelis/utl-make-new-column-with-the-previous-version-of-a-row-wps-hash-lag-and-r-code
    https://github.com/rogerjdeangelis/utl-multitasking-the-hash-for-a-very-fast-distinct-ids
    https://github.com/rogerjdeangelis/utl-no-need-for-sql-or-sort-merge-use-a-elegant-hash-excel-vlookup
    https://github.com/rogerjdeangelis/utl-only-keep-groups-without-duplicated-accounts-hash-sql
    https://github.com/rogerjdeangelis/utl-output-the-student-with-the-highest-grade-hash-defer-open
    https://github.com/rogerjdeangelis/utl-remove-duplicate-words-from-a-sentence-hash-solution
    https://github.com/rogerjdeangelis/utl-replicate-sets-of-rows-across-many-columns-elegant-hash
    https://github.com/rogerjdeangelis/utl-sas-fcmp-hash-stored-programs-python-r-functions-to-find-common-words
    https://github.com/rogerjdeangelis/utl-sharing-hash-storage-with-two-separate-datasteps-in-the-same-SAS-session
    https://github.com/rogerjdeangelis/utl-simple-example-of-a-hash-of-hashes-hoh-to-split_a-table
    https://github.com/rogerjdeangelis/utl-simplest-case-of-a-hash-or-sql-lookup
    https://github.com/rogerjdeangelis/utl-two-table-join-benchmarks-hash-sortmerge-keyindex-and-sasfile
    https://github.com/rogerjdeangelis/utl-two-techniques-for-a-persistent-hash-across-datasteps-and-procedures
    https://github.com/rogerjdeangelis/utl-using-a-hash-to-compute-cumulative-sum-without-sorting
    https://github.com/rogerjdeangelis/utl_benchmarks_hash_merge_of_two_un-sorted_data_sets_with_some_common_variables
    https://github.com/rogerjdeangelis/utl_hash_lookup_with_multiple_keys_nice_simple_example
    https://github.com/rogerjdeangelis/utl_hash_merge_of_two_un-sorted_data_sets_with_some_common_variables
    https://github.com/rogerjdeangelis/utl_hash_persistent
    https://github.com/rogerjdeangelis/utl_how_to_reuse_hash_table_without_reloading_sort_of
    https://github.com/rogerjdeangelis/utl_many_to_many_merge_in_hash_datastep_and_sql
    https://github.com/rogerjdeangelis/utl_nice_example_of_a_hash_of_hashes_by_paul_and_don
    https://github.com/rogerjdeangelis/utl_nice_hash_example_of_rolling_count_of_dates_plus-minus_2_days_of_current_date
    https://github.com/rogerjdeangelis/utl_select_ages_less_than_the_median_age_in_a_second_table_paul_dorfman_hash_solution
    https://github.com/rogerjdeangelis/utl_simple_one_to_many_join_using_SQL_and_datastep_hashes
    https://github.com/rogerjdeangelis/utl_simplified_hash_how_many_of_my_friends_are_in_next_years_math_class
    https://github.com/rogerjdeangelis/utl_using_a_hash_to_transpose_and_reorder_a_table
    https://github.com/rogerjdeangelis/utl_using_md5hash_to_create_checksums_for_programs_and_binary_files

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
