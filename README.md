# trabalho.py
Código do trabalho de progamação

# --- 1. IMPORTAÇÕES ---
# Importa as bibliotecas necessárias

import tkinter as tk
from tkinter import ttk  # 'ttk' oferece widgets com uma aparência mais moderna
from tkinter import messagebox
import datetime  # Para obter o ano atual e calcular a idade do imóvel
import csv       # Para salvar os dados no arquivo .csv 
import os        # Para verificar se o arquivo .csv já existe

# --- 2. DEFINIÇÃO DAS CLASSES ---

class Imovel:

    
    def __init__(self, metragem: float, ano_construcao: int, quartos: int, 
                 banheiros: int, tipo: str, bairro: str, vagas_garagem: int):

        self.metragem = metragem
        self.ano_construcao = ano_construcao
        self.quartos = quartos
        self.banheiros = banheiros
        self.tipo = tipo  # 'Casa' ou 'Apartamento'
        self.bairro = bairro
        self.vagas_garagem = vagas_garagem

    @property
    def idade(self) -> int:

        ano_atual = datetime.datetime.now().year
        idade_calculada = ano_atual - self.ano_construcao
        # Garante que a idade não seja negativa (caso o ano de construção seja no futuro)
        return max(0, idade_calculada)

# -----------------------------------------------

class CalculadoraValor:

    # Dicionário com valores base (R$/m²) fictícios, mas plausíveis, para bairros do RJ.
    # Usado como um atributo de classe (compartilhado por todas as instâncias).
    VALOR_M2_BAIRRO = {
        'Leblon': 23000.0,
        'Ipanema': 21000.0,
        'Botafogo': 11000.0,
        'Barra da Tijuca': 9000.0,
        'Recreio': 7000.0,
        'Tijuca': 6000.0,
        'Meier': 4000.0,
        'Campo Grande': 2500.0,
        'Bangu': 2000.0
    }

    def __init__(self):

        pass

    def estimar_valor(self, imovel: Imovel) -> float:

        
        # 1. Obter o valor base por m² do bairro
        # Usamos .get() para buscar o valor. Se o bairro não for encontrado,
        # ele retorna 0.0 (valor padrão) para evitar erros.
        valor_m2 = self.VALOR_M2_BAIRRO.get(imovel.bairro, 0.0)
        
        if valor_m2 == 0.0:
            # Se o bairro não estiver na lista, não é possível calcular
            raise ValueError(f"Bairro '{imovel.bairro}' não encontrado em nossa base de dados.")
            
        valor_base = imovel.metragem * valor_m2
        
        # 2. Calcular Fator de Idade
        # Depreciação de 0.3% ao ano (0.003), com piso de 0.7 (30% de depreciação máxima).
        idade_imovel = imovel.idade  # Usa a @property da classe Imovel
        fator_idade = max(0.7, (1.0 - (idade_imovel * 0.003)))
        
        # 3. Calcular Fator de Comodidades
        # +2% por quarto, +1% por banheiro, +4% por vaga de garagem
        fator_comodidades = 1.0 + (imovel.quartos * 0.02) + \
                                  (imovel.banheiros * 0.01) + \
                                  (imovel.vagas_garagem * 0.04)
        
        # 4. Calcular Fator de Tipo
        # 'Casa' tem um multiplicador de 1.05 (5% mais cara)
        fator_tipo = 1.05 if imovel.tipo == 'Casa' else 1.0
        
        # 5. Cálculo Final
        valor_final_estimado = valor_base * fator_idade * fator_comodidades * fator_tipo
        
        return valor_final_estimado

# -----------------------------------------------

class InterfaceGrafica:

    
    def __init__(self, root):
      
        self.root = root
        self.root.title("Calculadora de Valor Estimado de Imóvel (P2 - UFRJ)")
        self.root.geometry("450x450") # Define um tamanho inicial

        # Instancia a calculadora.
        # Esta é uma forma de INTERAÇÃO ENTRE CLASSES (Composição): 
        # A InterfaceGrafica "tem uma" CalculadoraValor.
        self.calculadora = CalculadoraValor()
        
        # Variáveis para armazenar a última avaliação (para o botão Salvar)
        self.ultimo_imovel: Imovel = None
        self.ultimo_valor: float = 0.0

        # Chama o método que cria todos os componentes (widgets) da tela
        self._criar_widgets()

    def _criar_widgets(self):
   
        
        # --- Configuração do Estilo ---
        # Define um estilo para os widgets ttk
        style = ttk.Style()
        style.configure('TLabel', font=('Helvetica', 10))
        style.configure('TButton', font=('Helvetica', 10, 'bold'))
        style.configure('TEntry', font=('Helvetica', 10))
        style.configure('TOptionMenu', font=('Helvetica', 10))
        style.configure('Resultado.TLabel', font=('Helvetica', 12, 'bold'), foreground='blue')

        # --- Frame Principal ---
        # Um 'frame' é um container para organizar outros widgets
        frame = ttk.Frame(self.root, padding="20 20 20 20")
        frame.pack(expand=True, fill=tk.BOTH) # Faz o frame preencher a janela

        # --- Título Interno ---
        ttk.Label(frame, text="Insira os Dados do Imóvel", 
                  font=('Helvetica', 14, 'bold')).grid(
                      row=0, column=0, columnspan=2, pady=10)

        # --- 1. Bairro (OptionMenu) ---
        ttk.Label(frame, text="Bairro:").grid(row=1, column=0, sticky=tk.W, pady=5)
        # Pega a lista de bairros diretamente da classe CalculadoraValor
        self.bairros_lista = list(CalculadoraValor.VALOR_M2_BAIRRO.keys())
        self.var_bairro = tk.StringVar(value=self.bairros_lista[0]) # Valor padrão
        bairro_menu = ttk.OptionMenu(frame, self.var_bairro, self.bairros_lista[0], *self.bairros_lista)
        bairro_menu.grid(row=1, column=1, sticky=tk.EW, pady=5)

        # --- 2. Tipo (OptionMenu) ---
        ttk.Label(frame, text="Tipo:").grid(row=2, column=0, sticky=tk.W, pady=5)
        self.tipos_lista = ['Apartamento', 'Casa']
        self.var_tipo = tk.StringVar(value=self.tipos_lista[0]) # Valor padrão
        tipo_menu = ttk.OptionMenu(frame, self.var_tipo, self.tipos_lista[0], *self.tipos_lista)
        tipo_menu.grid(row=2, column=1, sticky=tk.EW, pady=5)

        # --- 3. Metragem (Entry) ---
        ttk.Label(frame, text="Metragem (m²):").grid(row=3, column=0, sticky=tk.W, pady=5)
        self.entry_metragem = ttk.Entry(frame)
        self.entry_metragem.grid(row=3, column=1, sticky=tk.EW, pady=5)

        # --- 4. Ano de Construção (Entry) ---
        ttk.Label(frame, text="Ano de Construção:").grid(row=4, column=0, sticky=tk.W, pady=5)
        self.entry_ano = ttk.Entry(frame)
        self.entry_ano.grid(row=4, column=1, sticky=tk.EW, pady=5)

        # --- 5. Quartos (Entry) ---
        ttk.Label(frame, text="Quartos:").grid(row=5, column=0, sticky=tk.W, pady=5)
        self.entry_quartos = ttk.Entry(frame)
        self.entry_quartos.grid(row=5, column=1, sticky=tk.EW, pady=5)

        # --- 6. Banheiros (Entry) ---
        ttk.Label(frame, text="Banheiros:").grid(row=6, column=0, sticky=tk.W, pady=5)
        self.entry_banheiros = ttk.Entry(frame)
        self.entry_banheiros.grid(row=6, column=1, sticky=tk.EW, pady=5)

        # --- 7. Vagas de Garagem (Entry) ---
        ttk.Label(frame, text="Vagas de Garagem:").grid(row=7, column=0, sticky=tk.W, pady=5)
        self.entry_vagas = ttk.Entry(frame)
        self.entry_vagas.grid(row=7, column=1, sticky=tk.EW, pady=5)

        # --- Botão de Calcular ---
        btn_calcular = ttk.Button(frame, text="Estimar Valor", command=self._on_calcular_click)
        btn_calcular.grid(row=8, column=0, columnspan=2, pady=(20, 5), sticky=tk.EW)
        
        # --- Botão de Salvar (Opcional) ---
        btn_salvar = ttk.Button(frame, text="Salvar Avaliação", command=self._on_salvar_click)
        btn_salvar.grid(row=9, column=0, columnspan=2, pady=5, sticky=tk.EW)

        # --- Label de Resultado ---
        self.label_resultado = ttk.Label(frame, text="Valor Estimado: R$ -", style='Resultado.TLabel')
        self.label_resultado.grid(row=10, column=0, columnspan=2, pady=(15, 0))

        # Configura as colunas do grid para se expandirem igualmente
        frame.columnconfigure(1, weight=1)

    def _on_calcular_click(self):
        
        try:
            # --- 1. Coleta e Validação dos Dados ---
            # Coleta dados dos 'Entry' e converte para os tipos corretos (float/int)
            metragem = float(self.entry_metragem.get().replace(',', '.'))
            ano_construcao = int(self.entry_ano.get())
            quartos = int(self.entry_quartos.get())
            banheiros = int(self.entry_banheiros.get())
            vagas_garagem = int(self.entry_vagas.get())
            
            # Coleta dados dos 'OptionMenu'
            tipo = self.var_tipo.get()
            bairro = self.var_bairro.get()
            
            # Validação simples
            ano_atual = datetime.datetime.now().year
            if ano_construcao > ano_atual or ano_construcao < 1800:
                messagebox.showerror("Erro de Validação", 
                                     f"O ano de construção ({ano_construcao}) deve ser um valor válido.")
                return

            if metragem <= 0 or quartos < 0 or banheiros < 0 or vagas_garagem < 0:
                messagebox.showerror("Erro de Validação", 
                                     "Metragem, quartos, banheiros e vagas não podem ser negativos.")
                return

            # --- 2. Criação do Objeto Imovel ---
            # Aqui, a InterfaceGrafica CRIA um objeto da classe Imovel
            # com os dados coletados. [cite: 10]
            imovel_a_avaliar = Imovel(
                metragem=metragem,
                ano_construcao=ano_construcao,
                quartos=quartos,
                banheiros=banheiros,
                tipo=tipo,
                bairro=bairro,
                vagas_garagem=vagas_garagem
            )

            # --- 3. Cálculo do Valor (Interação de Classes) ---
            # A InterfaceGrafica CHAMA o método da Calculadora, 
            # passando o objeto 'imovel_a_avaliar' como parâmetro.
            valor_estimado = self.calculadora.estimar_valor(imovel_a_avaliar)

            # --- 4. Armazenamento local ---
            # Salva o imóvel e o valor para o botão "Salvar"
            self.ultimo_imovel = imovel_a_avaliar
            self.ultimo_valor = valor_estimado

            # --- 5. Exibição do Resultado ---
            # Formata o valor para o padrão brasileiro (R$ 1.234.567,89)
            resultado_formatado = f"R$ {valor_estimado:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".")
            self.label_resultado.config(text=f"Valor Estimado: {resultado_formatado}")

        except ValueError:
            # Captura erros de conversão (ex: digitar 'abc' em metragem)
            messagebox.showerror("Erro de Entrada", 
                                 "Por favor, verifique se todos os campos numéricos "
                                 "(metragem, ano, quartos, banheiros, vagas) "
                                 "estão preenchidos com números válidos.")
        except Exception as e:
            # Captura qualquer outro erro inesperado
            messagebox.showerror("Erro Inesperado", f"Ocorreu um erro: {e}")

    def _on_salvar_click(self):
      
        # Verifica se um cálculo já foi realizado
        if self.ultimo_imovel is None:
            messagebox.showwarning("Aviso", "Você precisa calcular um valor antes de salvar.")
            return

        NOME_ARQUIVO = 'avaliacoes.csv'
        
        # Define o cabeçalho do arquivo
        CABEÇALHO = ['bairro', 'tipo', 'metragem', 'ano_construcao', 
                     'quartos', 'banheiros', 'vagas_garagem', 'valor_estimado']

        # Prepara a linha de dados com base no último imóvel calculado
        im = self.ultimo_imovel
        linha_dados = [
            im.bairro,
            im.tipo,
            im.metragem,
            im.ano_construcao,
            im.quartos,
            im.banheiros,
            im.vagas_garagem,
            f"{self.ultimo_valor:.2f}" # Salva o valor formatado com 2 casas decimais
        ]

        try:
            # Verifica se o arquivo já existe para decidir se escreve o cabeçalho
            # 'os.path.exists' verifica a existência do arquivo
            arquivo_existe = os.path.exists(NOME_ARQUIVO)
            
            # Abre o arquivo no modo 'a' (append - adicionar ao final)
            # 'newline=""' é importante para o módulo 'csv' funcionar corretamente
            with open(NOME_ARQUIVO, mode='a', newline='', encoding='utf-8') as f:
                # Cria um objeto 'writer' do módulo csv
                writer = csv.writer(f)
                
                # Se o arquivo não existia, escreve o cabeçalho primeiro
                if not arquivo_existe:
                    writer.writerow(CABEÇALHO)
                
                # Escreve a linha de dados
                writer.writerow(linha_dados)
            
            messagebox.showinfo("Sucesso", f"Avaliação salva com sucesso no arquivo '{NOME_ARQUIVO}'!")
            
        except IOError as e:
            # Captura erros de permissão de escrita, etc.
            messagebox.showerror("Erro de Arquivo", f"Não foi possível salvar o arquivo: {e}")
        except Exception as e:
            messagebox.showerror("Erro Inesperado", f"Ocorreu um erro ao salvar: {e}")


# --- 3. EXECUÇÃO DO PROGRAMA ---

def main():
   
    root = tk.Tk()  # Cria a janela principal
    app = InterfaceGrafica(root) # Cria a instância da nossa classe de GUI
    root.mainloop() # Inicia o "loop principal" do Tkinter (mantém a janela aberta)

# A linha 'if __name__ == "__main__":' é uma convenção em Python.
# Ela garante que o código dentro dela (a função main()) só seja
# executado quando o arquivo é rodado diretamente, e não quando
# ele é importado por outro arquivo.
if __name__ == "__main__":
    main()
