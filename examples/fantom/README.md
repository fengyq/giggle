Get source code

    git clone https://github.com/samtools/htslib.git
    cd htslib
    autoheader
    autoconf
    ./configure --disable-bz2 --disable-lzma --enable-libcurl
    make -j 20
    export HTSLIB_ROOT=`pwd`
    cd ..

    git clone https://github.com/ryanlayer/giggle.git
    cd giggle
    make
    export GIGGLE_ROOT=`pwd`
    cd ..

Get database data and index

    mkdir fantom_data
    cd fantom_data
    wget http://fantom.gsc.riken.jp/5/datafiles/latest/extra/Enhancers/Human.sample_name2library_id.txt
    cat Human.sample_name2library_id.txt \
    | sed -e "s/, */-/g" \
    | sed -e "s/ /_/g" \
    | sed -e "s/(/-/g" \
    | sed -e "s/)/-/g" \
    | sed -e "s/:/-/g" \
    | sed -e "s/'//g" \
    | sed -e "s/\^//g" \
    | sed -e "s/\///g" \
    >  Human.sample_name2library_id.sanitized.txt

    wget http://fantom.gsc.riken.jp/5/datafiles/latest/extra/Enhancers/human_permissive_enhancers_phase_1_and_2_expression_count_matrix.txt.gz
    zcat human_permissive_enhancers_phase_1_and_2_expression_count_matrix.txt.gz \
    > human_permissive_enhancers_phase_1_and_2_expression_count_matrix.txt

    mkdir split
    python $GIGGLE_ROOT/examples/fantom/rename.py \
        Human.sample_name2library_id.sanitized.txt \
        human_permissive_enhancers_phase_1_and_2_expression_count_matrix.txt \
        split
    
    $GIGGLE_ROOT/scripts/sort_bed "split/*" split_sort/

    time $GIGGLE_ROOT/bin/giggle index \
        -i "split_sort/*gz" \
        -o split_sort_b \
        -f -s
    Indexed 11284790 intervals.

    real    0m16.771s
    user    0m15.860s
    sys     0m0.627s

