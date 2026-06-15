---
title: "x"
slug: "x" 
date: 2026-05-23
translationKey: "x"
categories: ["nx"]
math: true
draft: true
---

## Introdução

Sendo bem honesto, eu comecei esse lab sem nem saber onde iria chegar... Mas vou tentar falar sobre algumas coisas que eu julgo serem bem legais.

> Disclaimer: esse projeto não tem qualquer relação com o meu empregador

Nesse lab, eu vou utilizar algumas tecnologias para emular alguns comportamentos de uma rede de transporte e como eu implementaria algumas soluções voltadas a um ambiente de missão crítica.

Para isso temos alguns requisitos, temos 3 tipos de redes, que não devem falar entre si por questões de segurança e que cada uma tem uma finalidade única:
- Rede Corporativa
	- Voz
	- Video
	- Internet 
* Rede de TO (Automação/SPCS)
	* Relés
	* CLPs
	* Gateways
- Rede de Telecomunicações
	- Estrutura de base para os clientes internos das Redes Corporativa e TO

Para esse caso, vou usar uma VPN de camada 2 entre os switches que estão simulando um equipamento SDH (pseudowire).


