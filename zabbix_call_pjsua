# -*- coding: utf-8 -*-

import sys
import pjsua as pj

# Callback para receber eventos da conta SIP
class MyAccountCallback(pj.AccountCallback):
    def __init__(self, account):
        pj.AccountCallback.__init__(self, account)

    def on_reg_state(self):
        print("Registro SIP status: ", self.account.info().reg_status)

# Callback para receber eventos da chamada
class MyCallCallback(pj.CallCallback):
    def __init__(self, call):
        pj.CallCallback.__init__(self, call)
        self.wav_player_id = None

    def on_state(self):
        print("Estado da chamada: ", self.call.info().state_text,
              ", código:", self.call.info().last_code,
              "(", self.call.info().last_reason, ")")
        if self.call.info().state == pj.CallState.CONFIRMED:
            print("Chamada atendida. Reproduzindo áudio.")
            # Quando a chamada é atendida, tocar o arquivo de áudio
            try:
                self.wav_player_id = lib.create_player('arquivo_de_audio.wav', loop=False)
                player_slot = lib.player_get_slot(self.wav_player_id)
                call_slot = self.call.info().conf_slot
                lib.conf_connect(player_slot, call_slot)
            except pj.Error as e:
                print("Erro ao tocar o áudio:", str(e))

    def on_media_state(self):
        if self.call.info().media_state == pj.MediaState.ACTIVE:
            print("Mídia ativa - áudio está conectado.")
        elif self.call.info().media_state == pj.MediaState.DISCONNECTED:
            print("Mídia desconectada.")

# Inicializa a biblioteca pjsua
def log_cb(level, msg, length):
    print(msg)

# Configuração básica da biblioteca
lib = pj.Lib()

try:
    lib.init(log_cfg=pj.LogConfig(level=3, callback=log_cb))

    # Configuração do transporte UDP
    transport = lib.create_transport(pj.TransportType.UDP,
                                     pj.TransportConfig(5060))
    print("Transporte criado no endereço", transport.info().host)

    # Inicializa a biblioteca
    lib.start()

    # Desativa os dispositivos de áudio
    lib.set_null_snd_dev()

    # Cria a conta SIP com o login e senha do ramal 112
    account_config = pj.AccountConfig()
    account_config.id = "sip:ramal@sip"
    account_config.reg_uri = "sip:seu_sip"
    account_config.auth_cred = [pj.AuthCred("*", "ramal_login", "ramal@senha")]  # Aceitar qualquer domínio

    account = lib.create_account(account_config)
    account_cb = MyAccountCallback(account)

    # Aguarda até que o registro SIP seja completo
    print("Aguardando registro...")
    pj.time.sleep(2)


    # Inicia a chamada para o ramal 154
    call = account.make_call("sip:numero_destino@SIP", MyCallCallback(None))

    # Aguarda a chamada ser completada ou finalizar
    print("Realizando a chamada para o número externo..")
    pj.time.sleep(20)

    # Desliga a chamada após o tempo de teste
    call.hangup()

except pj.Error as e:
    print("Erro ao fazer a chamada:", str(e))
finally:
    lib.destroy()
    lib = None
