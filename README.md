# PROJECTE_03
#!/bin/bash

readonly ARCHIVE_DIR='/archive'

usage() {

  echo >&2
  echo "Usage: ${0} [-dra] USER [USERN]..." >&2
  echo 'Disable local user.' >&2
  echo '  -d  Delete accounts instead of disable them.' >&2
  echo '  -r  Delete the main directory associated to account.' >&2
  echo '  -a  Creates a file of the main directory assigned to the account.' >&2
  exit 1
}

if [[ $EUID -ne 0 ]]
then
echo -e "The user is not ROOT, so it is not allowed to run the script"
exit 1
fi


while getopts dra OPTION
do
  case ${OPTION} in
    d) DELETE_USER='true' ;;
    r) REMOVE_OPTION='true' ;;
    a) ARCHIVE='true' ;;
    ?) usage ;;
  esac
done

shift "$(( OPTIND - 1 ))"

if [[ "${#}" < 1 ]]
then
  usage
fi

for USERNAME in "${@}"
do
  echo "Processing user: ${USERNAME}"

  USERID=$(id -u ${USERNAME})
  if [[ "${USERID}" -lt 1000 ]]
  then
    echo "Refuse to remove the file ${USERNAME} user with UID ${USERID}." >&2
    exit 1
  fi


  if [[ "${ARCHIVE}" = 'true' ]]
  then

    if [[ ! -d "${ARCHIVE_DIR}" ]]
    then
      echo "Making directory ${ARCHIVE_DIR} ."
      mkdir -p ${ARCHIVE_DIR}

      if [[ "${?}" != 0 ]]
      then
        echo "File of direcotry ${ARCHIVE_DIR} is unable to create." >&2
        exit 1
      fi
    fi

    HOME_DIR="/home/${USERNAME}"
    ARCHIVE_FILE="${ARCHIVE_DIR}-${USERNAME}.tgz"
    if [[ -d "${HOME_DIR}" ]]
    then
      echo "Archiving ${HOME_DIR} in ${ARCHIVE_FILE}"
      tar -zcf ${ARCHIVE_FILE} ${HOME_DIR} &> /dev/null
      if [[ "${?}" != 0 ]]
      then
        echo "Unable to create ${ARCHIVE_FILE}." >&2
        exit 1
      fi
    else
       echo "${HOME_DIR} doesn't exist." >&2
       exit 1
    fi
  fi

#Deleting home directory 
	if [[ "${REMOVE_OPTION}" = 'true' ]]
  then
	rm -r /home/${USERNAME}
	echo "Directory deleted. "
	fi

#Deleting user
  if [[ "${DELETE_USER}" = 'true' ]]
  then

    userdel ${USERNAME}

    if [[ "${?}" != 0 ]]
    then
      echo "User ${USERNAME} has not been deleted." >&2
      exit 1
    fi
    echo "User ${USERNAME} has been deleted."
  else

    chage -E 0 ${USERNAME}


    if [[ "${?}" != 0 ]]
    then
      echo "User ${USERNAME} has not been disabled." >&2
      exit 1
    fi
    echo "User ${USERNAME} has been disabled."
  fi
done

exit 0
