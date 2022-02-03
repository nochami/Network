#!/bin/bash

endCol="\e[0m"
red="\033[0;31m"
green="\033[0;32m"
yellow="\033[0;33m"


interfaces=`ls /sys/class/net`

echo -e "Available$green interfaces:$endCol\n$green$interfaces$endCol"


function modify_network()
{
read -p "Which interface you want to modify? " interface

path_to_interface=/etc/sysconfig/network-scripts/ifcfg-$interface
if [[ `ls /sys/class/net | grep -w $interface` ]]
then
	bootproto=`grep "BOOTPROTO" "$path_to_interface" | cut -d "=" -f2`
	if [[ "$bootproto" ]]
	then
		read -p "BOOTPROTO=$bootproto do you want to change it? [y/N] " bootproto_action
		if [[ "$bootproto_action" =~ ^([yY])$ ]]
		then
			read -p "To which value you want to change it to? " new_bootproto
			sed -i s/"$bootproto"/"$new_bootproto"/g "$path_to_interface"
		fi
	else
		read -p "BOOTPROTO doesn't exist create? [y/N] " bootproto_action
		if [[ "$bootproto_action" =~ ^([yY])$ ]]
		then
			read -p "Provide value for BOOTPROTO: " new_bootproto
			echo "BOOTPROTO=$new_bootproto" >> "$path_to_interface"
		fi
	fi


	ipaddr=`grep "IPADDR" "$path_to_interface" | cut -d "=" -f2`
	if [[ "$ipaddr" ]]
	then
		read -p "IPADDR=$ipaddr do you want to change it? [y/N] " ipaddr_action
		if [[ "$ipaddr_action" =~ ^([yY])$ ]]
		then
			read -p "Provide new ip address for IPADDR: " new_ipaddr
			sed -i s/"$ipaddr"/"$new_ipaddr"/g "$path_to_interface"
		fi
	else
		read -p "IPADDR doesn't exist create? [y/N] " ipaddr_action
		if [[ "$ipaddr_action" =~ ^([yY])$ ]]
		then
			read -p "Provide ip address for IPADDR: " new_ipaddr
			echo "IPADDR=$new_ipaddr" >> "$path_to_interface"
		fi
	fi

	netmask=`grep "NETMASK" "$path_to_interface" | cut -d "=" -f2`
	if [[ "$netmask" ]]
	then
		read -p "NETMASK=$netmask do you want to change it [y/N] " netmask_action
		if [[ "$netmask_action" =~ ^([yY])$ ]]
		then
			read -p "Provide new ip for NETMASK: " new_netmask
			sed -i s/"$netmask"/"$new_netmask"/g "$path_to_interface"
		fi
	else
		read -p "NETMASK doesn't exist create? [y/N] " netmask_action
		if [[ "$netmask_action" =~ ^([yY])$ ]]
		then
			read -p "Provide ip address for NETMASK: " new_netmask
			echo "NETMASK=$new_netmask" >> "$path_to_interface"
		fi
	fi

	gateway=`grep "GATEWAY" "$path_to_interface" | cut -d "=" -f2`
	if [[ "$gateway" ]]
	then
		read -p "GATEWAY=$gateway do you want to change it? [y/N] " gateway_action
		if [[ "$gateway_action" =~ ^([yY])$ ]]
		then
			read -p "provide new ip for GATEWAY: " new_gateway
			sed -i s/"$gateway"/"$new_gateway"/g "$path_to_interface"
		fi
	else
		read -p "GATEWAY doesn't exist create? [y/N] " gateway_action
		if [[ "$gateway_action" =~ ^([yY])$ ]]
		then
			read -p "Provide ip address for GATEWAY: " new_gateway
			echo "GATEWAY=$new_gateway" >> "$path_to_interface"
		fi
	fi



	dns=`grep "DNS" "$path_to_interface" | cut -d "=" -f2`
	if [[ "$dns" ]]
	then
		echo -e "You already use this dns\n$green$dns$endCol"
	fi

	read -p "Want to add new DNS? [y/N] " dns_action
	if [[ "$dns_action" =~ ^([yY])$ ]]
	then
		read -p "How many DNS you want to add? " quantity

		for (( counter=1; counter<=$quantity; counter++ ))
		do
			read -p "Proivde ip address for DNS$counter: " dns_ip
			echo "DNS$counter=$dns_ip" >> "$path_to_interface"
		done

	fi

	# Restart network daemon
	read -e -i "y" -p "Restart network? [Y/n] " restart_prompt
	if [[ "$restart_prompt" =~ ^([yY])$ ]]
	then
		`systemctl restart network`
	fi

	# Show state of configuration file for interface
	echo -e "\n$green$interface configuration:$endCol"
	cat $path_to_interface
else
	echo -e "$red$interface not found!$endCol"
	setup_network

fi
}


function create_network_interface()
{
read -p "which name do you want to give to new interface ifcfg file? " interface_name
`cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-$interface_name`
path_to_interface=/etc/sysconfig/network-scripts/ifcfg-$interface_name

sed -i s/"UUID"/";UUID"/g "$path_to_interface"
ls /etc/sysconfig/network-scripts/
}


read -p $'Which action?\n1)Create settings file for interface\n2)Setup settings for existing interface file\n' action


case $action in
1) create_network_interface;;
2) modify_network;;
esac
