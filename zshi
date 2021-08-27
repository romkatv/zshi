#!/usr/bin/env zsh

emulate -L zsh -o no_unset

if (( ARGC == 0 )); then
  print -ru2 -- 'Usage: zshi <init-command> [zsh-flag]...

The same as plain `zsh [zsh-flag]...` except that an additional
<init-command> gets executed after all standard Zsh startup files
have been sourced.'
  return 1
fi

() {
  local init=$1
  shift
  local tmp
  {
    tmp=$(mktemp -d ${TMPDIR:-/tmp}/zsh.XXXXXXXXXX) || return
    local rc
    for rc in .zshenv .zprofile .zshrc .zlogin; do
      >$tmp/$rc <<<'{
  ZDOTDIR="$_zshi_zdotdir"
  if [[ -f "$ZDOTDIR/'$rc'" && -r "$ZDOTDIR/'$rc'" ]]; then
    "builtin" "source" "--" "$ZDOTDIR/'$rc'"
  fi
} always {
  if [[ -o "no_rcs" ||
        -o "login" && "'$rc'" == ".zlogin" ||
        -o "no_login" && "'$rc'" == ".zshrc" ||
        -o "no_login" && -o "no_interactive" && "'$rc'" == ".zshenv" ]]; then
    "builtin" "unset" "_zshi_rcs" "_zshi_zdotdir"
    "builtin" "command" "rm" "-rf" "--" '${(q)tmp}'
    "builtin" "eval" '${(q)init}'
  else
    _zshi_zdotdir=${ZDOTDIR:-~}
    ZDOTDIR='${(q)tmp}'
  fi
}' || return
    done
    _zshi_zdotdir=${ZDOTDIR:-~} ZDOTDIR=$tmp zsh "$@"
  } always {
    [[ -e $tmp ]] && rm -rf -- $tmp
  }
} "$@"
