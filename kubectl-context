#!/bin/bash

# Copyright (C) 2021 Sangkeun Lee - All Rights Reserved
# Permission to copy and modify is granted under the Creative Commons Attribution 4.0 license
# Last revised 2021-11-12
#
# References
#  https://github.com/ahmetb/kubectx
#  https://www.bughunter2k.de/blog/cursor-controlled-selectmenu-in-bash

if [[ "$1" == "init" ]]; then
    echo "export KUBECONFIG=\$(ls ~/.kube/cfg/* | xargs | tr ' ' ':')"
    exit 0
fi


KUBECTL=$(which kubectl)

function show_current() {
    current=$($KUBECTL config current-context)
    tput sgr0
    echo -n "Current context: "; tput setaf 5; echo "${current}"; tput sgr0
}

function name() {
    echo $1 | cut -d' ' -f1
}


function get_current() {
    ctx="$($KUBECTL config current-context)"
    for (( i=0; i < ${#menu[@]}; i++ )); do
        current=$(name ${menu[$i]})
        if [[ $current == $ctx ]]; then
            cur=$i
        fi
    done
}

function draw_menu() {
    for i in "${menu[@]}"; do
        if [[ ${menu[$cur]} == $i ]]; then
            tput setaf 5; echo  "*         $i"; tput sgr0;
        else
            tput sgr0; echo "          $i";
        fi
    done
}

function clear_menu() {
    for i in "${menu[@]}"; do tput cuu1; done
    # tput ed
}

function cleanup() {
    tput cnorm
}
trap cleanup EXIT



readarray -t menu < <($KUBECTL config get-contexts | tr  '*' ' ' | awk 'NR>1 {gsub(/^[ ]+/, ""); print $0}')

tput civis
cur=0
get_current

# draw head
head=$($KUBECTL config get-contexts | head -n1)
tput setaf 4; printf '%s\n'  "$head"; tput sgr0

draw_menu

while read -sn1 key; do # 1 char (not delimiter), silent
    # check for enter/space
    if [[ "$key" == "" ]]; then break; fi

    # catch multi-char special key sequences
    read -sN1 -t 0.0001 k1; read -sN1 -t 0.0001 k2; read -sN1 -t 0.0001 k3
    key+=${k1}${k2}${k3}

    case "$key" in
        # cursor up, left: previous item
        k|h|$'\e[A'|$'\e0A'|$'\e[D'|$'\e0D'|$'^P') (( cur > 0 )) && (( cur-- ));;
        # cursor down, right: next item
        l|j|$'\e[B'|$'\e0B'|$'\e[C'|$'\e0C'|$'^N') (( cur < ${#menu[@]}-1 )) && (( cur++ ));;
        # home: first item
        $'\e[1~'|$'\e0H'|$'\e[H') cur=0;;
        # end: last item
        $'\e[4~'|$'\e0F'|$'\e[F') (( cur=${#menu[@]}-1 ));;
        # q, esc: quit
        q|$'\e') echo "Abort"
                 show_current
                 exit -1;;
    esac

    # redraw menu
    clear_menu
    draw_menu
done

$KUBECTL config use-context $(name ${menu[$cur]}) > /dev/null
show_current
