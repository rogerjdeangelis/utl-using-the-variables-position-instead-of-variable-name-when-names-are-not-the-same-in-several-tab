# utl-using-the-variables-position-instead-of-variable-name-when-names-are-not-the-same-in-several-tab
Using the variables position instead of variable name when names are not the same in several tables
    Using the position of the variable instead of the variable name when names are not the same in several tables

    github
    https://tinyurl.com/r79utvp
    https://github.com/rogerjdeangelis/utl-using-the-variables-position-instead-of-variable-name-when-names-are-not-the-same-in-several-

    I need to write code that will depend only on a columns position, not the variable name.

    Suppose you receive monthly sas tables where the variable names change,
    but the position of the variables in the program data vector does not change.

    I need to keep the first column, drop columns 2 and 3 and compute BMI using columns 4 and 5.

    I know what the CORRECT names and positions for the five columns

    THREE SOLUTIONS

       1. Best sql union  (automatically uses the longest lengths)
            Bartosz Jablonski
            yabwon@gmail.com

       2. R  (most of the code is getting data in and out of R in dataframes)

       3. Array and varlist macros


    SAS Forum
    related to
    https://tinyurl.com/wy3vhdt
    https://communities.sas.com/t5/New-SAS-User/How-do-I-select-a-range-of-variables-using-an-quot-if-quot/m-p/628908

    related to
    github
    https://tinyurl.com/yb9znnrk
    https://github.com/rogerjdeangelis/utl_using_position_of_specifc_values_in_one_array_as_indexes_to_second_array

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;


    data jan;
      set sashelp.class(obs=3
            rename=(name=student sex=gender age=ageyears height=hgt weight=wgt));
    run;quit;

    data feb;
      set sashelp.class(firstobs=16
         rename=(name=surname sex=mf age=ageyrs height=hgt_in weight=wgt_lbs));
    run;quit;

    Note diff names same position;

    WORK.JAN total obs=3

    Obs    STUDENT    GENDER    AGEYEARS     HGT     WGT

     1     Joyce        F          11       51.3    50.5
     2     Louise       F          12       56.3    77.0
     3     Alice        F          13       56.5    84.0

    WORK.FEB total obs=4

    Obs    SURNAME    MF    AGEYRS    HGT_IN    WGT_LBS

     1     William    M       15       66.5      112.0
     2     Ronald     M       15       67.0      133.0
     3     Alfred     M       14       69.0      112.5
     4     Philip     M       16       72.0      150.0


    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

    * DROP SEX and AGE and compute BMI;
    * rename but the 'wrong' names are not known in advance';

    Up to 40 obs WORK.YEAR_TO_DATE total obs=7

    Obs     NAME      HEIGHT    WEIGHT      BMI

     1     Joyce       51.3       50.5    13.4900
     2     Louise      56.3       77.0    17.0777
     3     Alice       56.5       84.0    18.4986
     4     William     66.5      112.0    17.8045
     5     Ronald      67.0      133.0    20.8285
     6     Alfred      69.0      112.5    16.6115
     7     Philip      72.0      150.0    20.3414

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __  ___
    / __|/ _ \| | | | | __| |/ _ \| '_ \/ __|
    \__ \ (_) | | |_| | |_| | (_) | | | \__ \
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/
               _               _
     ___  __ _| |  _   _ _ __ (_) ___  _ __
    / __|/ _` | | | | | | '_ \| |/ _ \| '_ \
    \__ \ (_| | | | |_| | | | | | (_) | | | |
    |___/\__, |_|  \__,_|_| |_|_|\___/|_| |_|
            |_|
    ;

    * sql automatically aligns columns with the union operator;
    proc sql;

      create
         table year_to_date as
      select
         name
        ,height
        ,weight
        ,703 * weight / (height * height) as bmi
      from

        ( select
            *
         from
            class
         union
            all
         select
            *
         from
            jan
         union
            all
         select
            *
         from
            feb )
    ;quit;

    *____
    |  _ \
    | |_) |
    |  _ <
    |_| \_\

    ;
    * put tables where R can reach them;
    libname sd1 "d:/sd1";
    proc copy in=work out=sd1;
    select jan feb;
    run;quit;
    libname sd1clear;

    %utl_submit_r64('
    library(haven);
    library(SASxport);
    jan<-read_sas("d:/sd1/jan.sas7bdat");
    feb<-read_sas("d:/sd1/feb.sas7bdat");
    janfeb<-data.frame(Map(c,jan,feb));
    colnames(janfeb)=c("name","sex","age","height","weight");
    janfeb$bmi=703 * janfeb$weight / janfeb$height^2;
    janfeb<-janfeb[,-(2:3)];
    janfeb[] <- lapply(janfeb, function(x) if(is.factor(x)) as.character(x) else x);
    write.xport(janfeb,file="d:/xpt/janfeb.xpt");
    ');

    libname xpt xport "d:/xpt/janfeb.xpt";
    data janfeb;
      set xpt.janfeb;
    run;quit;
    libname xpt clear;


    *                                             _ _     _
      __ _ _ __ _ __ __ _ _   _  __   ____ _ _ __| (_)___| |_
     / _` | '__| '__/ _` | | | | \ \ / / _` | '__| | / __| __|
    | (_| | |  | | | (_| | |_| |  \ V / (_| | |  | | \__ \ |_
     \__,_|_|  |_|  \__,_|\__, |   \_/ \__,_|_|  |_|_|___/\__|
                          |___/
    ;

    * just in case;
    %arraydelete(jan);
    %arraydelete(feb);

    %array(jan,values=%varlist(jan));
    %array(feb,values=%varlist(feb));

    data year_to_date;
       set
          jan (drop=&jan2 &jan3 rename=(&jan1=name  &jan4=height &jan5=weight))
          feb (drop=&feb2 &feb3 rename=(&feb1=name  &feb4=height &feb5=weight))
       ;

       bmi=703 * weight / (height * height);

    run;quit;

