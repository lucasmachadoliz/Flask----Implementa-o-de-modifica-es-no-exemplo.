from flask import Flask, request, jsonify

app = Flask(__name__)

# Lista para armazenar os dados do sensor e a temperatura
banco_em_memoria = []


# Rota para receber a leitura do sensor (Método POST)
@app.route('/sensores/clima', methods=['POST'])
def registrar_leitura():
    dados = request.get_json()

    # Adiciona a temperatura informada à nossa lista
    if 'id' not in dados or 'temperatura' not in dados:
        return jsonify({"erro": "Os campos 'id' e 'temperatura' são obrigatórios."}), 400

    banco_em_memoria.append(dados)

    return jsonify({"mensagem": "Leitura registrada com sucesso!", "dados": dados}), 201


# Rota para consultar o resumo das leituras (Método GET)
@app.route('/sensores/clima', methods=['GET'])
def listar_todos():

    # TODO 3 - Filtro por Query Parameter
    acima_de = request.args.get('acima_de')

    if acima_de is not None:
        try:
            acima_de = float(acima_de)
        except ValueError:
            return jsonify({"erro": "O parâmetro 'acima_de' deve ser um número."}), 400

        leituras_filtradas = [
            leitura for leitura in banco_em_memoria
            if leitura['temperatura'] > acima_de
        ]

        return jsonify({
            "total_registros": len(leituras_filtradas),
            "leituras": leituras_filtradas
        }), 200

    return jsonify({
        "total_registros": len(banco_em_memoria),
        "leituras": banco_em_memoria
    }), 200


# Rota para buscar um sensor específico pelo ID
@app.route('/sensores/clima/<sensor_id>', methods=['GET'])
def buscar_por_id(sensor_id):
    for leitura in banco_em_memoria:
        if leitura['id'] == sensor_id:
            return jsonify(leitura), 200

    return jsonify({
        "erro": f"Sensor '{sensor_id}' não encontrado."
    }), 404


# TODO 2 - Implementação do Método PUT
@app.route('/sensores/clima/<sensor_id>', methods=['PUT'])
def atualizar_sensor(sensor_id):

    dados = request.get_json()

    if 'temperatura' not in dados:
        return jsonify({
            "erro": "O campo 'temperatura' é obrigatório."
        }), 400

    for leitura in banco_em_memoria:
        if leitura['id'] == sensor_id:
            leitura['temperatura'] = dados['temperatura']

            return jsonify({
                "mensagem": "Temperatura atualizada com sucesso!",
                "dados": leitura
            }), 200

    return jsonify({
        "erro": f"Sensor '{sensor_id}' não encontrado."
    }), 404


# Rota para remover um sensor específico da lista
@app.route('/sensores/clima/<sensor_id>', methods=['DELETE'])
def deletar_sensor(sensor_id):
    global banco_em_memoria

    tamanho_original = len(banco_em_memoria)

    banco_em_memoria = [
        leitura for leitura in banco_em_memoria
        if leitura['id'] != sensor_id
    ]

    if len(banco_em_memoria) < tamanho_original:
        return jsonify({
            "mensagem": f"Sensor '{sensor_id}' removido com sucesso."
        }), 200

    return jsonify({
        "erro": f"Sensor '{sensor_id}' não encontrado para exclusão."
    }), 404


def emTeste():
    pass


if __name__ == '__main__':
    app.run(debug=True)
