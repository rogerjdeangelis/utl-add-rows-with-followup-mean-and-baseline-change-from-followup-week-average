# utl-add-rows-with-followup-mean-and-baseline-change-from-followup-week-average
Add rows with followup mean and baseline change from followup week average
    Add rows with followup mean and baseline change from followup week average

    Week 1is baseline weeks 2-4 are followup.
    No hardcoding in this solution

    GitHub
    https://tinyurl.com/7bdr85cy
    https://github.com/rogerjdeangelis/utl-add-rows-with-followup-mean-and-baseline-change-from-followup-week-average


    https://tinyurl.com/zkk47pc
    https://communities.sas.com/t5/SAS-Programming/Calculation-across-rows/m-p/724273

    Tools
    https://tinyurl.com/n8bea8tx
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    libname sd1 "d:/sd1";
    options validvarname=upcase;

    data sd1.have;
    infile cards expandtabs;
    input WEEK $ Units : comma12.;
    format units comma12.;
    cards4;
    Week1 81,855
    Week2 80,009
    Week3 71,972
    Week4 68,035
    ;;;;
    run;quit;


    Up to 40 obs SD1.HAVE total obs=4

    Obs    WEEK     UNITS

     1     Week1    81855
     2     Week2    80009
     3     Week3    71972
     4     Week4    68035

    ----------------------------------------------------------------
    RULES ADD ROWS 5 an 6
    ----------------------------------------------------------------

     5    AVG_R2_4  mean(have(rows=2:4,col=2))  (80009+71972+68035)/3

     6    PctDif    100*(have(row=1,col=2) - AVG_R2_4)/AVG_R2_4
          Basline   100*(81855-73338.67)/73338.67

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;

    * incase you reun or data exists;
    %utlfkil(d:/xpt/want.xpt);

    proc datasets lib=work nolist;
    delete want;
    run;quit;

    * drop down to R;
    %utl_submit_r64('
      library(haven);
      library(SASxport);
      library(data.table);
      mat=as.matrix(read_sas("d:/sd1/have.sas7bdat")[,2]);
      avg<-mean(mat[2:nrow(mat),1]);
      var<- 100*(mat[1,1] - avg)/avg;
      want<-data.table(WEEK=c("AVG_R24","VARIANCE"), UNITS= c(avg,var) );
      want;
      write.xport(want,file="d:/xpt/want.xpt");
    ');

    libname xpt xport "d:/xpt/want.xpt";
    data want;
      set sd1.have
          xpt.want
      ;
    run;quit;

    proc print data=xpt.want;
    run;quit;

    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

    Up to 40 obs from WANT total obs=6

    Obs    WEEK           UNITS

     1     Week1       81855.00
     2     Week2       80009.00
     3     Week3       71972.00
     4     Week4       68035.00
     5     AVG_R24     73338.67
     6     VARIANCE       11.61

