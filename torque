#!/usr/bin/env bash
#
# Torque - minimal tui for transmission-daemon.

refresh() {
    printf '\e[?7l\e[?25l\e[2J\e[H'
    shopt -s checkwinsize; (:;:)
    [[ -z "$LINES" ]] && read -r LINES COLUMNS < <(stty size)
    ((j=(LINES-2)/3))
}

status() {
    printf '\e[2m\e[%s;H%s\e[m' "$((LINES-1))" "$1"
}

get_torrents() {
    IFS=$'\n' read -d "" -ra t < <(transmission-remote -l)
    unset 't[0]' 't[-1]' 2>/dev/null

    t=("${t[@]//[0-9] [a-z][a-z][a-z]?/.}")
    t=("${t[@]//Up & Down/Active}")
    t=("${t[@]//Downloading/Active}")
    t=("${t[@]//     None/0 MB}")

    for((i=${k:=0};i<(j=j>${#t[@]}?${#t[@]}:j);i++));{ t_print "${t[i]/n\/a/0}";}
    status "[s]tart [p]ause [r]emove [o]pen [j/k] [q]uit scroll ($j/${#t[@]})"$'\e[H'
}

t_print() {
    IFS=" %" read -r num perc have unit _ up down _ stat name <<< "$1"

    ((size=perc!=0?${have/.*}*100/perc:0,c=perc==100?33271340:31000000))

    printf '\e[K\e[2m%s\e[m \e[1m\e[%s%b%s\e[m\n' \
           "$num:" "${c:0:2}m" "\\u${c:2:4}\\${c:6}" "$name"
    printf '\e[K\e7%s\e8\e[14C%s\e8\e[32C%s\e8\e[42C%s\e8\e[52C%s\n\e[K\n' \
           "   $stat: " "$have / $size $unit" "(${perc}%)" "⇣ $down" "⇡ $up"
}

prompt() {
    send() { transmission-remote "$@" >/dev/null; }
    status $'\e[B\e[?25h'

    case "$1" in
        s) read -rp "start torrent: #";  send -t "$REPLY" -s ;;
        p) read -rp "pause torrent: #";  send -t "$REPLY" -S ;;
        r) read -rp "remove torrent: #"; send -t "$REPLY" -r; k=0 ;;
        o) read -rp "load magnet: ";     send -a "$REPLY"; k=0 ;;
        j) ((j==${#t[@]}))||((k=k>=j?k:++k,j=j<${#t[@]}?++j:j)) ;;
        k) ((k==0))||((k=k<=j?k>0?--k:0:j,j=j>0?--j:j)) ;;
        q) exit 0 ;;
    esac

    [[ "$1" =~ (j|k) ]] || refresh && printf '\e[?25l\e[H'
}

main() {
    refresh

    trap $'printf \e[?25h\e[?7h\e[999B' EXIT
    trap  'refresh; k=0' SIGWINCH

    for ((;;)); { get_torrents; read -rsN1 -t1 && prompt "$REPLY"; }
}

main
