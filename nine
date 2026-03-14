#!/bin/env bash

PGNAME="tec-nine"
VERSION="v0.0.1"
COMMAND=
TASKDIR=
PGNDIR="$HOME/.local/lib/tec/pgn"
BRDNAME=
PRJNAME=
TASKNAME=
ISDEBUG=false

# TODO: add option for config path on CLI
CONFIG_FILE=

function _check_args()
{
    # TODO: check board name when support is added
    if [ -z "$PGNDIR" ]; then
        elog "no plugin directory is passed"
        exit 1
    elif [ -z "$TASKDIR" ]; then
        elog "no task directory is passed"
        exit 1
    fi
}

function find_config()
{
    declare -a cfgs=(
        "$HOME/.tec/pgn/nine.json"
        "$HOME/.config/tec/pgn/nine.json"
    )

    for cfg in "${cfgs[@]}"; do
        if [ -f "$cfg" ]; then
            CONFIG_FILE="$cfg"
            break
        fi
    done
}

function elog()
{
    echo "$PGNAME:" "$@" >&2
}

function dlog()
{
    if [ "$ISDEBUG" == true ]; then
        echo "$PGNAME:" "$@"
    fi
}

function _nine_init()
{
    find_config
}


function usage()
{
    cat << EOF
Usage: tec nine COMMAND [OPTION]... [ARGS]...
    Tec plugin manager

    COMMAND list:
      del       delete plugin
      help      show this help message and exit
      list      list plugins and statuses
      sync      update plugin
      ver       show version and exit

    Use 'tec nine help COMMAND' to get help on command
EOF
}

function nine_help()
{
    local command="$1"
    declare -a commands=("add" "del" "help" "list" "ver")

    for cmd in "${commands[@]}"; do
        if [ "$cmd" = "$command" ]; then
            eval "_help_$cmd"
            return
        fi
    done
    usage
}

# TODO: add option -f : delete and reinstall plugin again
# TODO: compile/install each plugin if needed
# TODO: Add support to pass repo URL as an option
function nine_sync()
{
    declare -A repocfg

    [ -z "$CONFIG_FILE" ] && exit 0
    REPOS="$(jq -c ".repos[]" "$CONFIG_FILE")"

    while read -r repo; do
        while IFS="=" read -r key val; do
            [[ -n "$key" ]] && repocfg["$key"]="$val"  # Skip empty lines
            #echo "-- pgnunits: $key -> $val"
        done < <(jq -r 'to_entries[] | "\(.key)=\(.value)"' <<< "$repo")
        repocfg["name"]="$(basename ${repocfg["link"]} ".git")"
        repocfg["name"]=${repocfg["name"]#tec-}

        # check if repo is already cloned
        if [ -d "$PGNDIR/${repocfg["name"]}" ]; then
            echo "${repocfg["name"]}[OK]: repo already cloned"
        else
            if git clone -q --recursive "${repocfg["link"]}" "$PGNDIR/${repocfg["name"]}" 2>/dev/null; then
                echo "${repocfg["name"]}[OK]: repo is cloned"
            else
                echo "${repocfg["name"]}[ERR]: no such repo '${repocfg["link"]}'"
            fi
        fi

        # Pull changes for each repo
        if git -C "$PGNDIR/${repocfg["name"]}" pull --quiet "${repocfg["link"]}" 2>/dev/null; then
            echo "${repocfg["name"]}[OK]: changes pulled"
        else
            echo "${repocfg["name"]}[ERR]: could not pull changes"
        fi
        echo
    done <<< "$REPOS"
}

function nine_del()
{
    local pgname="$1"

    printf "Are you sure to delete pluign in %s? [y/N] " "$PGNDIR/$pgname"
    read -r choice
    if [ "$choice" != "Y" ] && [ "$choice" != "y" ] && [ "$choice" != "yes" ]; then
        echo "plugin deletion is canceled"
        exit 1
    fi

    if [ -z "$pgname" ]; then
        elog "no plugin is provided. Try 'nine help' for more info"
        exit 1
    elif [ ! -d "$PGNDIR/$pgname" ]; then
        elog "$pgname: no such plugin. Try 'nine help' for more info"
        exit 1
    else
        echo "delete plugin in $PGNDIR/$pgname"
        rm -rf "$PGNDIR/$pgname"
    fi
}

# TODO: add stutus: repo is cloned or not
function nine_list()
{
    find "$PGNDIR/" -maxdepth 1 -type d ! -path "$PGNDIR/" | while read -r pgn; do
        local name="${pgn##*/}"
        local desc="tec plugin"

        [ -s "$PGNDIR/$name/desc" ] && desc="$(cat "$PGNDIR/$name/desc")"
        printf "%-6s - %s\n" "$name" "$desc"
    done
}

function nine_ver()
{
    echo "$PGNAME: $VERSION"
}


OPTS=$(getopt -o b:d:i:p:P:T:h --long board:,debug:prj:,taskid:,pgndir:,taskdir:help -n "$PGNAME" -- "$@")
if [ $? -ne 0 ]; then
    #echo "error parsing options" >&2
    exit
fi

## Reset the positional parameters to the parsed options
eval set -- "$OPTS"

while true; do
    case "$1" in
        -b)
            BRDNAME="$2"
            shift 2
            ;;
        -d)
            ISDEBUG="$2"
            shift 2
            ;;
        -i)
            TASKNAME="$2"
            shift 2
            ;;
        -p)
            PRJNAME="$2"
            shift 2
            ;;
        -P)
            PGNDIR="$2"
            shift 2
            ;;
        -T)
            TASKDIR="$2"
            shift 2
            ;;
        --)
            shift
            break
            ;;
        *)
            echo "$PGNAME: invalid option '$1'" >&2
            exit 1
    esac
done

COMMAND="$1"; shift 1


_check_args
find_config
[ -f "$CONFIG_FILE" ] && jq empty "$CONFIG_FILE" || exit 1

if [ "$COMMAND" = "del" ]; then
    nine_del "$@"
elif [ "$COMMAND" = "help" ]; then
    nine_help "$@"
elif [ -z "$COMMAND" ] || [ "$COMMAND" = "list" ]; then
    nine_list "$@"
elif [ "$COMMAND" = "sync" ]; then
    nine_sync "$@"
elif [ "$COMMAND" = "ver" ]; then
    nine_ver "$@"
else
    elog "'$COMMAND': no such command"
fi
