
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract OdontoPro {
    struct Paciente {
        string nombre;
        uint servicioActivo;
        uint ultimaConsulta;
    }

    struct Servicio {
        string descripcion;
        uint precio;
        bool activo;
    }

    address public owner;
    mapping(address => Paciente) public pacientes;
    mapping(uint => Servicio) public servicios;

    uint public servicioCounter;

    modifier soloOwner() {
        require(msg.sender == owner, "No autorizado");
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function registrarServicio(string memory descripcion, uint precio) public soloOwner {
        servicioCounter++;
        servicios[servicioCounter] = Servicio(descripcion, precio, true);
    }

    function registrarPaciente(address paciente, string memory nombre) public soloOwner {
        pacientes[paciente] = Paciente(nombre, 0, block.timestamp);
    }

    function asignarServicio(address paciente, uint servicioId) public soloOwner {
        require(servicios[servicioId].activo, "Servicio inactivo");
        pacientes[paciente].servicioActivo = servicioId;
        pacientes[paciente].ultimaConsulta = block.timestamp;
    }

    function pagarServicio(address paciente) public payable {
        uint servicioId = pacientes[paciente].servicioActivo;
        require(servicios[servicioId].precio == msg.value, "Monto incorrecto");
        // fondo recibido por owner (clínica)
    }

    function obtenerDatosPaciente(address paciente) public view returns (string memory nombre, string memory servicio, uint fecha) {
        Paciente memory p = pacientes[paciente];
        Servicio memory s = servicios[p.servicioActivo];
        return (p.nombre, s.descripcion, p.ultimaConsulta);
    }
}
