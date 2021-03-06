# utl_many_to_many_merge_in_hash_datastep_and_sql
Problem join on subject where dose_date &lt;= date.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Problem many to many join on subject where dose_date <= date

      PSUEDO CODE

       select from
          dose and  ae datasets
       where
          dose.subject  equal ae.subject    and
          dose.dose_date less than or equal ae.date

       First three programs gave the same results in WPS and SAS.
       WPS does not support curobs (yet?);

       FOUR SOLUTIONS
       --------------

           1. SQL
           2. Datastep
           3. Hash Paul Dorfman sashole@bellsouth.net
           4. Hash explicit rid and curobs Bart Jablonski <yabwon@GMAIL.COM>
              WPS "ERROR: Found "curobs" when expecting one of END, KEY, NOBS or POINT"

    github
    https://tinyurl.com/yazu3uyn
    https://github.com/rogerjdeangelis/utl_many_to_many_merge_in_hash_datastep_and_sql

    SAS-L
    https://listserv.uga.edu/cgi-bin/wa?A2=SAS-L;d5646af5.1807a

    INPUT
    =====

     WORK.AE total obs=2

      SUBJECT      DATE      AE

       10001      21347    hyping
       10001      21357    Peeing

     WORK.DOSE total obs=3

                  DOSE_
      SUBJECT     DATE    AMOUNT

       10001     21215    10 mg
       10001     21243    20 mg
       10001     21357    30 mg


    EXAMPLE OUTPUT
    --------------

     WORK.WANT total obs=5

                                      DOSE_
        SUBJECT     DATE      AE       DATE    AMOUNT

         10001     21347    hyping    21215    10 mg
         10001     21347    hyping    21243    20 mg

         10001     21357    Peeing    21215    10 mg
         10001     21357    Peeing    21243    20 mg
         10001     21357    Peeing    21357    30 mg

     RULES

       Here is a outer join

       WORK.HAVSQL total obs=6

                                     DOSE_         |  RULE
       SUBJECT      AE     AMOUNT    DATE   DATE   |  dose_date <= date
                                                   |
        10001     hyping   10 mg    21215  21347   |
        10001     hyping   20 mg    21243  21347   |
        10001     hyping   30 mg    21357  21347   |  ** drop this record
                                                   |     because code_date > date
        10001     Peeing   10 mg    21215  21357   |
        10001     Peeing   30 mg    21357  21357   |
        10001     Peeing   20 mg    21243  21357   |


    PROCESS
    =======

        1. SQL

           proc sql;
             create
                 table havSQL as
             select
                 l.*
                ,r.date
                ,r.ae
             from
                dose as l, ae as r
             where
                l.subject   = r.subject and
                l.dose_date le r.date

           ;quit;

        2. Datastep

           data want;

                set dose nobs=obs_dose;
                set ae end=dne1 nobs=obs_ae;

                do pt_dose=1 to obs_dose;
                   set dose point=pt_dose;
                   do pt_ae=1 to obs_ae;
                      set ae point=pt_ae;
                      if dose_date le date then output ;
                   end;
                end;

                stop;

           run;quit;

        3. HASH

           data want ;
             if _n_ = 1 then do ;
               dcl hash h (multidata:"y") ;
               h.definekey ("subject") ;
               h.definedata ("dose_date", "rid") ;
               h.definedone () ;
               do rid = 1 by 1 until (z) ;
                 set dose end = z ;
                 h.add() ;
               end ;
             end ;
             set ae ;
             *do while (h.do_over() = 0) ; /*9.4 and up*/
             if h.find() = 0 then do until (h.find_next() ne 0) ;
               if date < dose_date then continue ;
               set dose point = rid ;
               output ;
             end ;
           run ; quit ;

        4. HASH  (explicit rid curobs)

           data want_bart ;
             if _n_ = 1 then do ;
               dcl hash h (multidata:"y") ;
               h.definekey ("subject") ;
               h.definedata ("dose_date", "rid") ;
               h.definedone () ;
               do until (z) ;  /* <--- no explicit RID loop */
                 set dose
                   /* (where = (<some additional conditions on "dose" here>))*/
                   end = z curobs=RID;
                   /*<--- curobs here */
                 h.add() ;
               end ;
             end ;
             set ae ;
           *do while (h.do_over() = 0) ; /*9.4 and up*/
             if h.find() = 0 then do until (h.find_next() ne 0) ; /*9.2 and up*/
               if date < dose_date then continue ;
               set dose point = rid ;
               output ;
             end ;
           run ;

    OUTPUT
    ======


    Up to 40 obs WORK.HAVSQL total obs=4

                      DOSE_
    Obs    SUBJECT     DATE    AMOUNT     DATE      AE

     1      10001     21215    10 mg     21347    hyping
     2      10001     21215    10 mg     21357    Peeing
     3      10001     21243    20 mg     21347    hyping
     4      10001     21243    20 mg     21357    Peeing

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;


    data ae;
      input subject    AE :$8.       date :date9. ;
      format date date9.;
    cards4;
    10001     hyping     12JUN2018
    10001     Peeing     22JUN2018
    ;;;;
    run;quit;

    data dose;
      input subject  Dose_date :date9.   Amount :&$5.;
      format dose_date date9.;
    cards4;
    10001    31JAN2018       10 mg
    10001    28FEB2018       20 mg
    10001    22JUN2018       30 mg
    ;;;;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    * see process for SAS code;


    * WPS;
    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";
     proc sql;
       create
           table wantSQL as
       select
           l.*
          ,r.date
          ,r.ae
       from
          wrk.dose as l, wrk.ae as r
       where
          l.subject   = r.subject and
          l.dose_date le r.date
     ;quit;
     proc print;
     run;quit;
     data want_datastep;
          set wrk.dose nobs=obs_dose;
          set wrk.ae end=dne1 nobs=obs_ae;

          do pt_dose=1 to obs_dose;
             set wrk.dose point=pt_dose;
             do pt_ae=1 to obs_ae;
                set wrk.ae point=pt_ae;
                if dose_date le date then output ;
             end;
          end;
          stop;
     run;quit;
     proc print;
     run;quit;
     data want_hash ;
       if _n_ = 1 then do ;
         dcl hash h (multidata:"y") ;
         h.definekey ("subject") ;
         h.definedata ("dose_date", "rid") ;
         h.definedone () ;
         do rid = 1 by 1 until (z) ;
           set wrk.dose end = z ;
           h.add() ;
         end ;
       end ;
       set wrk.ae ;
       *do while (h.do_over() = 0) ; /*9.4 and up*/
       if h.find() = 0 then do until (h.find_next() ne 0) ;
         if date < dose_date then continue ;
         set wrk.dose point = rid ;
         output ;
       end ;
     run ; quit ;
     proc print;
     run;quit;
     data want_bart ;
       if _n_ = 1 then do ;
         dcl hash h (multidata:"y") ;
         h.definekey ("subject") ;
         h.definedata ("dose_date", "rid") ;
         h.definedone () ;
         do until (z) ;
           set wrk.dose
             end = z curobs=RID;
           h.add() ;
         end ;
       end ;
       set wrk.ae ;
     *do while (h.do_over() = 0) ; /*9.4 and up*/
       if h.find() = 0 then do until (h.find_next() ne 0) ; /*9.2 and up*/
         if date < dose_date then continue ;
         set wrk.dose point = rid ;
         output ;
       end ;
     run ;
    ');

