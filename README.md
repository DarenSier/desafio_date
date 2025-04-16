import textwrap

def exibir_menu():
    menu = """
    ================ MENU ================
    [d]\tDepositar
    [s]\tSacar
    [e]\tExtrato
    [nc]\tNova conta
    [lc]\tListar contas
    [nu]\tNovo usuário
    [q]\tSair
    => """
    return input(textwrap.dedent(menu))

def depositar(saldo, valor, extrato, /):
    if valor > 0:
        saldo += valor
        extrato += f"Depósito:\tR$ {valor:.2f}\n"
        print("\n=== Depósito realizado com sucesso! ===")
    else:
        print("\n@@@ Valor inválido para depósito. @@@")
    return saldo, extrato

def sacar(*, saldo, valor, extrato, limite, saques_realizados, limite_saques):
    if valor > saldo:
        print("\n@@@ Saldo insuficiente. @@@")
    elif valor > limite:
        print("\n@@@ Limite de saque excedido. @@@")
    elif saques_realizados >= limite_saques:
        print("\n@@@ Limite diário de saques atingido. @@@")
    elif valor > 0:
        saldo -= valor
        extrato += f"Saque:\t\tR$ {valor:.2f}\n"
        saques_realizados += 1
        print("\n=== Saque realizado com sucesso! ===")
    else:
        print("\n@@@ Valor inválido para saque. @@@")
    return saldo, extrato

def exibir_extrato(saldo, /, *, extrato):
    print("\n================ EXTRATO ================")
    print(extrato if extrato else "Não foram realizadas movimentações.")
    print(f"\nSaldo:\t\tR$ {saldo:.2f}")
    print("==========================================")

def criar_usuario(usuarios):
    cpf = input("Informe o CPF (somente número): ")
    if filtrar_usuario(cpf, usuarios):
        print("\n@@@ Já existe usuário com esse CPF! @@@")
        return

    nome = input("Nome completo: ")
    nascimento = input("Data de nascimento (dd-mm-aaaa): ")
    endereco = input("Endereço (logradouro, nro - bairro - cidade/UF): ")

    usuarios.append({
        "nome": nome,
        "data_nascimento": nascimento,
        "cpf": cpf,
        "endereco": endereco
    })

    print("=== Usuário criado com sucesso! ===")

def filtrar_usuario(cpf, usuarios):
    return next((u for u in usuarios if u["cpf"] == cpf), None)

def criar_conta(agencia, numero, usuarios):
    cpf = input("Informe o CPF do usuário: ")
    usuario = filtrar_usuario(cpf, usuarios)

    if usuario:
        print("\n=== Conta criada com sucesso! ===")
        return {"agencia": agencia, "numero_conta": numero, "usuario": usuario}
    print("\n@@@ Usuário não encontrado. Criação de conta encerrada. @@@")

def listar_contas(contas):
    for conta in contas:
        print("=" * 100)
        print(textwrap.dedent(f"""
            Agência:\t{conta['agencia']}
            C/C:\t\t{conta['numero_conta']}
            Titular:\t{conta['usuario']['nome']}
        """))

def main():
    AGENCIA_PADRAO = "0001"
    LIMITE_SAQUES = 3
    LIMITE_VALOR_SAQUE = 500

    saldo = 0
    extrato = ""
    saques_realizados = 0
    usuarios = []
    contas = []

    while True:
        opcao = exibir_menu()

        if opcao == "d":
            valor = float(input("Valor do depósito: "))
            saldo, extrato = depositar(saldo, valor, extrato)

        elif opcao == "s":
            valor = float(input("Valor do saque: "))
            saldo, extrato = sacar(
                saldo=saldo,
                valor=valor,
                extrato=extrato,
                limite=LIMITE_VALOR_SAQUE,
                saques_realizados=saques_realizados,
                limite_saques=LIMITE_SAQUES
            )
            if valor > 0 and valor <= saldo and valor <= LIMITE_VALOR_SAQUE and saques_realizados < LIMITE_SAQUES:
                saques_realizados += 1

        elif opcao == "e":
            exibir_extrato(saldo, extrato=extrato)

        elif opcao == "nu":
            criar_usuario(usuarios)

        elif opcao == "nc":
            numero_conta = len(contas) + 1
            conta = criar_conta(AGENCIA_PADRAO, numero_conta, usuarios)
            if conta:
                contas.append(conta)

        elif opcao == "lc":
            listar_contas(contas)

        elif opcao == "q":
            print("\nSaindo do sistema. Até logo!")
            break

        else:
            print("\n@@@ Opção inválida. Tente novamente. @@@")

if __name__ == "__main__":
    main()
