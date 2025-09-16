-- LocalScript em StarterGui (modificado para suportar até 1500 e início customizado)

local Players = game:GetService("Players")
local TextChatService = game:GetService("TextChatService")
local player = Players.LocalPlayer

local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "ContadorGUI"
gui.ResetOnSpawn = false

-- Janela
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 260, 0, 220)
frame.Position = UDim2.new(0.3, 0, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
frame.Active = true
frame.Draggable = true

-- Caixa de número final
local caixaNumero = Instance.new("TextBox", frame)
caixaNumero.PlaceholderText = "Número final (1 a 1500)"
caixaNumero.Size = UDim2.new(0.9, 0, 0.15, 0)
caixaNumero.Position = UDim2.new(0.05, 0, 0.05, 0)
caixaNumero.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
caixaNumero.Text = ""
caixaNumero.ClearTextOnFocus = false

-- Botões
local btnStart = Instance.new("TextButton", frame)
btnStart.Size = UDim2.new(0.4, 0, 0.15, 0)
btnStart.Position = UDim2.new(0.05, 0, 0.25, 0)
btnStart.Text = "Começar"
btnStart.BackgroundColor3 = Color3.fromRGB(100, 255, 100)

local btnStop = Instance.new("TextButton", frame)
btnStop.Size = UDim2.new(0.4, 0, 0.15, 0)
btnStop.Position = UDim2.new(0.55, 0, 0.25, 0)
btnStop.Text = "Parar"
btnStop.BackgroundColor3 = Color3.fromRGB(255, 100, 100)

-- Cooldown
local caixaCooldown = Instance.new("TextBox", frame)
caixaCooldown.PlaceholderText = "Cooldown em segundos (ex: 1)"
caixaCooldown.Size = UDim2.new(0.9, 0, 0.15, 0)
caixaCooldown.Position = UDim2.new(0.05, 0, 0.45, 0)
caixaCooldown.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
caixaCooldown.Text = "1"
caixaCooldown.ClearTextOnFocus = false

local cooldownLabel = Instance.new("TextLabel", frame)
cooldownLabel.Size = UDim2.new(0.9, 0, 0.07, 0)
cooldownLabel.Position = UDim2.new(0.05, 0, 0.60, 0)
cooldownLabel.BackgroundTransparency = 1
cooldownLabel.Text = "Cooldown = tempo entre falas"
cooldownLabel.TextScaled = true

-- Caixa início customizado
local caixaInicio = Instance.new("TextBox", frame)
caixaInicio.PlaceholderText = "Começar de qual número?"
caixaInicio.Size = UDim2.new(0.9, 0, 0.15, 0)
caixaInicio.Position = UDim2.new(0.05, 0, 0.72, 0)
caixaInicio.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
caixaInicio.Text = ""
caixaInicio.ClearTextOnFocus = false

-- Conversor de número para extenso (1..1500)
local units = { [1]="UM",[2]="DOIS",[3]="TRÊS",[4]="QUATRO",[5]="CINCO",[6]="SEIS",[7]="SETE",[8]="OITO",[9]="NOVE" }
local teens = { [10]="DEZ",[11]="ONZE",[12]="DOZE",[13]="TREZE",[14]="QUATORZE",[15]="QUINZE",[16]="DEZESSEIS",[17]="DEZESSETE",[18]="DEZOITO",[19]="DEZENOVE" }
local tens = { [20]="VINTE",[30]="TRINTA",[40]="QUARENTA",[50]="CINQUENTA",[60]="SESSENTA",[70]="SETENTA",[80]="OITENTA",[90]="NOVENTA" }
local hundreds = { [100]="CENTO",[200]="DUZENTOS",[300]="TREZENTOS",[400]="QUATROCENTOS",[500]="QUINHENTOS",[600]="SEISCENTOS",[700]="SETECENTOS",[800]="OITOCENTOS",[900]="NOVECENTOS" }

local function under100(n)
	if n == 0 then return "" end
	if n <= 9 then return units[n] end
	if n >= 10 and n <= 19 then return teens[n] end
	local t = math.floor(n/10)*10
	local u = n % 10
	if u == 0 then return tens[t] else return tens[t] .. " E " .. units[u] end
end

local function under1000(n)
	if n == 0 then return "" end
	if n == 100 then return "CEM" end
	if n < 100 then return under100(n) end
	local h = math.floor(n/100)*100
	local rest = n % 100
	local hname = hundreds[h] or ""
	if rest == 0 then return hname else return hname .. " E " .. under100(rest) end
end

local function numberToWords(n)
	if n <= 0 then return "ZERO" end
	if n < 1000 then return under1000(n) end
	local thousands = math.floor(n/1000)
	local rest = n % 1000
	local prefix = (thousands == 1) and "MIL" or (numberToWords(thousands) .. " MIL")
	if rest == 0 then return prefix
	elseif rest < 100 then return prefix .. " E " .. numberToWords(rest)
	else return prefix .. " " .. numberToWords(rest) end
end

local rodando = false
local canal = TextChatService.TextChannels:WaitForChild("RBXGeneral")

local function falar(msg)
    canal:SendAsync(msg)
end

btnStart.MouseButton1Click:Connect(function()
	if rodando then return end
	local numeroFinal = tonumber(caixaNumero.Text)
	local tempo = tonumber(caixaCooldown.Text)
	local numeroInicio = tonumber(caixaInicio.Text) or 1
	
	if not numeroFinal or numeroFinal < 1 or numeroFinal > 1500 then
		caixaNumero.Text = "Digite de 1 a 1500"
		return
	end
	if not tempo or tempo <= 0 then
		caixaCooldown.Text = "Cooldown inválido"
		return
	end
	if numeroInicio < 1 or numeroInicio > numeroFinal then
		caixaInicio.Text = "Início inválido"
		return
	end
	
	rodando = true
	task.spawn(function()
		for i = numeroInicio, numeroFinal do
			if not rodando then break end
			local palavra = numberToWords(i) or tostring(i)
			falar(palavra .. "!")
			task.wait(tempo)
		end
		rodando = false
	end)
end)

btnStop.MouseButton1Click:Connect(function()
	rodando = false
end)
