<script>
    import { each } from "svelte/internal";
    import Admin from "./Admin.svelte";
    import Button from "./Parts/Button.svelte";
    import LoginInformationPrompt from "./Parts/LoginInformationPrompt.svelte";
    import AutomaticDevice from "./Parts/AutomaticDevice.svelte";
    import found_devices from "./stores/found_devices.js";
    import LoginInformation from "./stores/LoginInformation";

    // constants
    const ngrok_url = "https://29b2-38-110-15-66.ngrok-free.app"

    let foundDevices = [];

    
    found_devices.subscribe((data) => {foundDevices = data})
    const maxInputs = 3
    $:found_num_devices = foundDevices.length;
    $:selected_num_devices = devices.length;
    $:found_scrollable = found_num_devices > maxInputs;
    $:selected_scrollable = selected_num_devices > maxInputs


    const add_device = (device) =>
    {
        devices = [...devices, device];

        foundDevices = foundDevices.filter(item => item !== device);
    }

    const removeSelectedDevice = (device) =>
    {
        console.log(device);
        devices = devices.filter(item => item !== device);
        foundDevices = [...foundDevices, device]
    }

    const handleToggleDevices = () =>
    {
        automatic_devices = !automatic_devices;
        devices = [];
    }

    const handleToggleRouter = () => {
        console.log(devices)
        devices = devices.map(device => {
      return { ...device, type: device.type === '' ? 'Router' : (device.type === 'Router' ? 'switch' : 'Router') };
    });
    console.log(devices)
    }

    // variables
    let automatic_devices = true;
    let devices = []; // Stores objects containing the device information
    let supported_device_types = ["Switch", "Router"];
    let number_of_devices = 3;
    let supported_vendors = [{name:"Cisco", operatingSystems:["cisco_ios", "cisco_ios_telnet", "NX-OS", "IOS-XR"]}, {name:"Juniper", operatingSystems:["1", "2", "3"]}];
    let toggleManual = true;
    let login_information = {}
    LoginInformation.subscribe(data => login_information = data);

    let manual_script = "";
    let config_map = 
    {
        'interface config': {interface_name: '', ip_address: '', subnet_mask: '', description: ''},
       'static routing': {destination_network: '', next_hop: '', subnet_mask: ''},
       'dynamic routing': {routing_protocol: '', network: '', subnet_mask: '', interface: '', areas: '', ospf_leader: false},
       'vlan config': {vlan_id: '', vlan_name: '', vlan_description: '', delete: false, vlan_state: true, vlan_mtu: '-1', vlan_ip_address: '', vlan_ip_mask: '', vlan_tagged_interfaces: [], vlan_untagged_interfaces: []},
       'vtp config': {vtp_domain: '', vtp_password: ''},

    }
    
    // stores data from the configuration tool to be handled by the python script

    let tool_config_script = config_map["interface config"]; 
    let config_type = "interface config";
    let interface_to_add = "";
    let untagged_interface_to_add

    const handleToggleManual = () => {
        toggleManual = !toggleManual;
    }

    const handle_config_type = (event) => {
        tool_config_script = config_map[event.target.value];
        config_type = event.target.value;
    }

    const add_tagged_interface = () => {
        console.log("Adding interface" + interface_to_add);
        if(interface_to_add !== "") tool_config_script.vlan_tagged_interfaces = [...tool_config_script.vlan_tagged_interfaces,interface_to_add];
    }

    const add_untagged_interface = () => {
        if(untagged_interface_to_add !== "") tool_config_script.vlan_untagged_interfaces = [...tool_config_script.vlan_untagged_interfaces, untagged_interface_to_add];
    }

    $: scrollable = devices.length > maxInputs;

    // functions for form
    const addDevice = () => {
    devices = [...devices, {ip: "", type: devices.length > 0 ? devices[0].type : "Switch", vendor: "", os: ""}];
}

const removeDevice = (index) => {
    console.log(`Device removed: Device IP: ${devices[index].ip} Type: ${devices[index].type} Vendor: ${devices[index].vendor} OS: ${devices[index].os}`);
    // if(devices.length > 1)
     devices = devices.filter((_, i) => i !== index);
}

const handleVendorChange = (selectedVendorName) => {
    selectedVendor = supported_vendors.find(vendor => vendor.name === selectedVendorName);
}

const validateIP = (ip) => {
    const regex = /^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;
     if(ip !== "") return regex.test(ip);
    return false

}


//validate fields are correct and send to python script for implementation
async function submitHandler() {
  if (devices.length === 0) {
    alert("Please enter at least one device!");
  } else {
    console.log("Devices: " + JSON.stringify(devices) + "Configuration: " + JSON.stringify(tool_config_script));
    const url = `http://10.11.12.26:5000/handle_form_data`;
    const requestData = {
      "login_information": login_information,
      "devices": devices,
      "tool_config_script": toggleManual ? manual_script : tool_config_script,
      "manual": toggleManual,
      "config_type": config_type
    };

    const request = new Request(url, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(requestData),
    });

    try {
      const response = await fetch(request);
      const responseData = await response.json();
      if (responseData.success) {
        const message = responseData.message;
        const data = responseData.data;
        // display the message and result to the user
        console.log("Logging result:");
        console.log(message, " ", data);
      } else {
        const errorMessage = responseData.message;
        const errorData = responseData.data;
        console.error(errorMessage, " ", errorData);
      }
    } catch (error) {
      console.log("HTTP Request failed: Printing Error:");
      console.error(error);
      console.log("failed");
    }
  }
}

    
const logClick = () => {
    toggleManual = !toggleManual;
    console.log("Field is now: " + toggleManual);
}

const logInput = () => {
    console.log(manual_script);
}

// functions for handling configuration tool

let selectedVendor = supported_vendors[0];
</script>

<main>
    <div class=info-div>
    </div>
    <div class=devices>
        <form on:submit|preventDefault={submitHandler}>
            <LoginInformationPrompt/>
            {#if automatic_devices}
                <div class="automatic_devices">
                    <p>Discovered Devices</p>
                    <div class={found_scrollable ? "scroll" : ""}>
                        {#each foundDevices as device}
                            <AutomaticDevice {device}/>
                            <Button on:click={() => add_device(device)}>Select Device</Button>
                        {/each}
                    </div>
                    <div class="automatic_devices">
                    <p>Selected Devices</p>
                        <div class={selected_scrollable ? "scroll" : ""}>

                        {#each devices as device}
                            <AutomaticDevice {device}/>
                            <Button on:click={() => removeSelectedDevice(device)}>Remove Device</Button>
                        {/each}
                        </div>
                    </div>
                </div>
                {:else}
            <p>Please input the devices you wish to configure</p>
            <div class={scrollable ? "container" : ""}>
                {#each devices as device, i}
                <div class="input">
                    <label for="address">
                        Device IP:
                        <input type="text" bind:value={device.ip} placeholder="IP Address">
                        {#if validateIP(device.ip)}
                        <p style="color: green;" class="ip-valid">Valid IP</p>
                        {:else}
                        <p style="color: red;" class="ip-valid">Invalid IP</p>
                        {/if}
                    </label>
                    <label for="type">
                        Device Type:
                        <select name="type" id="type" bind:value={device.type}>
                            {#each supported_device_types as device}
                            <option value="{device}">{device}</option>
                            {/each}
                        </select>
                    </label>
                    <label for="vendor">
                        Device Vendor:
                        <select name="vendor" id="vendor" bind:value={device.vendor} on:change={() => handleVendorChange(device.vendor)}>
                            {#each supported_vendors as vendor}
                            <option value="{vendor.name}">{vendor.name}</option>
                            {/each}
                        </select>
                    </label>
                    <label for="os">
                        Operating System:
                        <select name="operatingSystem" id="operatingSystem" bind:value={device.os}>
                            {#each selectedVendor.operatingSystems as os}
                            <option value="{os}">{os}</option>
                            {/each}

                        </select>

                    </label>
                    <Button on:click={() => removeDevice(i)} small={true} >Remove Device</Button>

                </div>
                {/each}
            </div>
            {/if}
            <Button on:click={addDevice}>Add Device</Button>
            <label for="tool">
                <Button on:click={handleToggleManual}> Toolbar</Button>
                <Button on:click={handleToggleDevices}>Toggle Discovery</Button>
                {#if !toggleManual}
                <Button on:click={handleToggleRouter}>Toggle Device Type</Button>

                {/if}
            </label>
            <br>
            {#if toggleManual}
                <textarea name="script" id="manual_script" bind:value={manual_script} placeholder="Input configuration script here"></textarea>
            {:else}
                <div class=tool-div>
                    <h2>Configuration Tool</h2>
                    <!-- Prompt router configuration options -->
                    {#if devices.length === 0}
                        <h2>Please select a device to configure</h2>
                        {:else }


                    {#if devices[0].type === 'Router'} 
                    <p>Routing</p>
                    Conifiguration type:
                        <select name="config-type" id="switch-config-type" on:change={handle_config_type}>
                            <option  value="interface config">Interface Configuration</option>
                            <option value="static routing">Static Routing</option>
                            <option value="dynamic routing">Dynamic Routing</option>
                        </select> <br><br>
                    <!-- Interface configuration -->
                    {#if config_type === "interface config"}
                    <p>Interface</p>
                    <label for="Interface-Name">
                        Interface Name:
                        <input type="text" bind:value={tool_config_script.interface_name}>
                    </label><br>
                    <label for="Interface-ip">
                        Interface IP:
                        <input type="text" bind:value={tool_config_script.ip_address}>
                    </label><br>
                    <label for="Subnet-Mask">
                        Subnet Mask:
                        <input type="text" bind:value={tool_config_script.subnet_mask}>
                    </label><br>
                    <label for="Interface-Description">
                        Description:
                        <input type="text" bind:value={tool_config_script.description}>
                    </label><br>
                    
                    {:else if config_type === "static routing"}
                    <label for="Static-Destination-network">
                        Destination Network:
                        <input type="text" bind:value={tool_config_script.destination_network}>
                    </label><br>
                    <label for="static-next-hop">
                        next hop:
                        <input type="text" bind:value={tool_config_script.next_hop}>
                    </label><br>
                    <label for="subnet_mask">
                        Subnet Mask:
                        <input type="text" bind:value={tool_config_script.subnet_mask}>
                    </label><br>
                    {:else if config_type === "dynamic routing"}
                    <label for="routing-protocol">
                        Routing Protocol:
                        <select name="config-type" id="switch-config-type" bind:value={tool_config_script
                        .routing_protocol}>
                            <option  value="RIPv1">RIPv1</option>
                            <option value="RIPv2">RIPv2</option>
                            <option value="OSPF">OSPF</option>
                            <option value="EIGRP">EIGRP</option>
                        </select>
                    </label><br>
                    <label for="network">
                        IP Address:
                        <input type="text" bind:value={tool_config_script.network}>
                    </label><br>
                    <label for="subnet_mask">
                        Subnet Mask:
                        <input type="text" bind:value={tool_config_script.subnet_mask}>
                    </label><br>
                    <label for="interface">
                        Interface:
                        <input type="text" bind:value={tool_config_script.interface}>
                    </label><br>
                    {#if tool_config_script.routing_protocol === "OSPF"}
                    <label for="area">
                        Area:
                        <input type="text" bind:value={tool_config_script.areas}>
                    </label><br>
                    <label for="OSPF-Leader">
                        OSPF Leader:
                        <input type="checkbox" bind:value={tool_config_script.areas}>
                    </label><br>
                    {:else if tool_config_script.routing_protocol === "EIGRP"}
                    <label for="AS">
                        AS Number:
                        <input type="text" bind:value={tool_config_script.areas}>
                    </label><br>
                    {/if}
                    <!-- 'dynamic routing': {routing_protocol: '', network: '', subnet_mask: '', interface: '', areas: '', ospf_leader: false}, -->
                   
                    {/if}


                    <!-- Prompt switch configuration options -->
                    {:else}

                    <p>Switching</p>
                    <label for="config-type">
                        Conifiguration type:
                        <select name="config-type" id="switch-config-type" on:change={handle_config_type}>
                            <option  value="interface config">Interface Configuration</option>
                            <option value="vlan config">VLAN Configuration</option>
                            <option value="vtp config">VTP Configuration</option>
                        </select>
                    </label>
                    <!-- Interface configuration -->
                    {#if config_type === "interface config"}
                    <p>Interface</p>
                    <!-- {interface_name: '', ip_address: '', subnet_mask: '', description: ''} -->
                    <label for="Interface-Name">
                        Interface Name:
                        <input type="text" bind:value={tool_config_script.interface_name}>
                    </label><br>
                    <label for="Interface-ip">
                        Interface IP:
                        <input type="text" bind:value={tool_config_script.ip_address}>
                    </label><br>
                    <label for="Subnet-Mask">
                        Subnet Mask:
                        <input type="text" bind:value={tool_config_script.subnet_mask}>
                    </label><br>
                    <label for="Interface-Description">
                        Description:
                        <input type="text" bind:value={tool_config_script.description}>
                    </label><br>


                        <!-- VLAN configuration -->
                    {:else if config_type === "vlan config"}
                    <p>VLAN</p>
                    <!-- Deleting? -->
                    <p>Delete VLAN? <input type="checkbox" bind:checked={tool_config_script.delete}></p>
                    <br>
                        <!-- VLAN ID -->
                        <label for="VLAN-ID">
                            VLAN ID:
                            <input type="number" bind:value={tool_config_script.vlan_id}>
                        </label><br>
                        {#if !tool_config_script.delete}
                        <label for="VLAN-Name">
                        VLAN Name:
                        <input type="text" bind:value={tool_config_script.vlan_name}>
                    </label>
                    <br>
                    <label for="VLAN-Description">
                        VLAN Description:
                        <input type="number" bind:value={tool_config_script.vlan_description}>
                    </label><br>
                    <label for="VLAN-State">
                        Active or Suspended state?:
                        <input type="checkbox" bind:value={tool_config_script.vlan_state}>
                    </label><br>
                    <label for="VLAN-MTU">
                        VLAN MTU:
                        <input type="number" bind:value={tool_config_script.vlan_mtu}>
                    </label><br>
                    <label for="VLAN-IP">
                        VLAN IP:
                        <input type="number" bind:value={tool_config_script.vlan_ip_address}>
                    </label> <br>
                    <label for="VLAN-Mask">
                        VLAN Mask:
                        <input type="number" bind:value={tool_config_script.vlan_ip_mask}>
                    </label> <br>
                    <label for="VLAN-ID">
                        VLAN ID:
                        <input type="number" bind:value={tool_config_script.vlan_id}>
                    </label>
                    <br><br>
                    <div class="list-wrapper">
                        <p>Tagged Interfaces:</p>
                            {#each tool_config_script.vlan_tagged_interfaces as face}
                            {face}<br>
                            {/each}        
                    </div> <br>
                    <label for="VLAN-Tagged-Interfaces">
                        Add Tagged Interface:
                        <input type="text" bind:value={interface_to_add}>
                        <button type="button" on:click={add_tagged_interface}>Add Interface</button>
                    </label>
                    <br><br>
                    <div class="list-wrapper">
                        <p>Untagged Interfaces:</p>
                        {#each tool_config_script.vlan_untagged_interfaces as face}
                            {face}<br>
                        {/each}
                    </div> <br>
                    <label for="VLAN-Untagged-Interfaces">
                        Add Untagged Interface:
                        <input type="text" bind:value={untagged_interface_to_add}>
                        <button type="button" on:click={add_untagged_interface}>Add Interface</button>
                    </label>
                    {/if}
        
                    <!-- VTP configuration -->
                    {:else if config_type === "vtp config"}
                    <p>VTP</p>
                        <!-- 'vtp config': {vtp_domain: '', vtp_password: ''} -->
                        <label for="VTP-Domain">
                            VTP Domain:
                            <input type="text" bind:value={tool_config_script.vtp_domain}>
                        </label><br>
                        <label for="VTP-password">
                            VTP Password:
                            <input type="text" bind:value={tool_config_script.vtp_password}>
                        </label>
                    {/if}



                    {/if}
                    {/if}
                </div>
            {/if}
            <div class="button-wrapper">
                <Button  on:click={submitHandler} end={true}>Configure!</Button>
            </div>
            </form>
        </div>

</main>


<style>
*{
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    /*color: white;*/
}

p
{
    color: white;
}
label{
    text-align: start;
}

.list-wrapper{
    width: 100%;
    justify-content: center;
}

ul{
    width: 200px;
    border: solid 1px black
}

h1 h2 h3 h4{
    font-family: Georgia, 'Times New Roman', Times, serif;
}

main{
    justify-content: center;
}

label input{
    margin-bottom: 15px;
}

button{
    color: white;
    background-color: #00789D;
    border: none;
    border-radius: 10px;
}

.info-div{
    text-align: center;
}

.devices{
    display: flex;
    justify-content: center;
    align-items: center;
}

.automatic_devices
{
    margin-top: 5%;
    padding: 20px;
    min-width: 50%;
    min-height: 20em;
    align-content: center;
    background-color: #162B3F;

}

.input{
    display: flex;
    height: 50px;
    background-color: #162B3F;
    margin: 10px;
    padding-top: 15px;
    width: 100%;
}

.toolbar-ip-valid{
    width: 100px;
    background-color: #162B3F;
    align-self: center;
}

.ip-valid{
    transform: translate(120px, -30px);
}

.input button{
    margin-right: 5px;
    height: 20px;
}
.input label{
    margin: 0 5px;
}

@media screen and (max-width: 1125px){
    .input{
        flex-direction: column;
        height: 200px;
        margin-bottom: 5%;

    }
    .input button{
        width: 6em;
        height: 35px;
        padding-top: 0;
        margin: 15px;
    }
}

.button-wrapper{
    margin: 20px 10px;
    text-align: end;
}

#manual_script{
    resize: none;
    width: 100%;
    height: 40rem;
    align-self: center;
    background-color: #162B3F;
    border: none;
    caret-color: white;
    cursor: pointer;
    color: white;
}

.container{
    width: 100%;
    max-height: 250px;
    overflow-y: auto;
    overflow-x: hidden;
    padding-right: 16px;
}

.tool-div{
    color: white;
    background-color: #162B3F;
    border: solid 1px black;
    text-align: center;
}

.scroll{
    width: 100%;
    max-height: 265px;
    overflow-y: auto;
    overflow-x: hidden;
    padding-right: 16px;
}
</style>