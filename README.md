if not game:IsLoaded() then game.Loaded:Wait() end
local ZenithUI = {}
local uis, ts = game:GetService("UserInputService"), game:GetService("TweenService")

function Create(typestring, instance, props)
    local obj = typestring == "Instance.new" and Instance.new(instance) or typestring == "Drawing.new" and Drawing.new(instance)
    if not obj then return nil end
    for i, v in pairs(props or {}) do pcall(function() obj[i] = v end) end
    return obj
end

local Themes = {
    Dark = {
        Background = Color3.fromRGB(30, 30, 30),
        Background_ = Color3.fromRGB(45, 45, 45),
        Line = Color3.fromRGB(82, 82, 82),
        TextLabel = Color3.fromRGB(240, 240, 240),
        TextLabel_ = Color3.fromRGB(124, 124, 124),
    },
}

function ZenithUI:MakeWindow(options)
    local window = {}
    window.Title = options.Name or "Zenith UI"
    window.HidePremium = options.HidePremium or false
    window.SaveConfig = options.SaveConfig or false
    window.ConfigFolder = options.ConfigFolder or "ZenithConfig"
    window.Tabs = {}

    -- UI Setup
    local theme = Themes["Dark"]
    local sg = Create("Instance.new", "ScreenGui", {Name = "ZenithUI", Parent = game.CoreGui})
    local mainFrame = Create("Instance.new", "Frame", {Size = UDim2.new(0, 500, 0, 400), BackgroundColor3 = theme.Background, Parent = sg})
    local tabContainer = Create("Instance.new", "Frame", {Size = UDim2.new(1, 0, 0, 40), BackgroundColor3 = theme.Background_, Parent = mainFrame})

    function window:MakeTab(tabOptions)
        local tab = {}
        tab.Name = tabOptions.Name
        tab.Elements = {}

        -- Tab Button
        local tabButton = Create("Instance.new", "TextButton", {Text = tab.Name, Size = UDim2.new(0, 100, 1, 0), Parent = tabContainer})

        -- Tab Frame
        local tabFrame = Create("Instance.new", "Frame", {Size = UDim2.new(1, 0, 1, -40), Position = UDim2.new(0, 0, 0, 40), BackgroundColor3 = theme.Background, Visible = #window.Tabs == 0, Parent = mainFrame})

        function tab:AddButton(btnOptions)
            local button = Create("Instance.new", "TextButton", {Text = btnOptions.Name, Size = UDim2.new(0, 150, 0, 40), BackgroundColor3 = theme.Background_, Parent = tabFrame})
            button.MouseButton1Click:Connect(btnOptions.Callback)
            return button
        end

        function tab:AddToggle(toggleOptions)
            local toggle = false
            local button = tab:AddButton({Name = toggleOptions.Name, Callback = function()
                toggle = not toggle
                toggleOptions.Callback(toggle)
            end})
            return button
        end

        function tab:AddSlider(sliderOptions)
            local sliderFrame = Create("Instance.new", "Frame", {Size = UDim2.new(0, 200, 0, 50), BackgroundColor3 = theme.Background_, Parent = tabFrame})
            local slider = Create("Instance.new", "TextButton", {Size = UDim2.new(0, 20, 0, 20), BackgroundColor3 = theme.TextLabel, Parent = sliderFrame})

            slider.MouseButton1Down:Connect(function()
                local conn
                conn = uis.InputChanged:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseMovement then
                        local percent = math.clamp((input.Position.X - sliderFrame.AbsolutePosition.X) / sliderFrame.AbsoluteSize.X, 0, 1)
                        slider.Position = UDim2.new(percent, -10, 0.5, -10)
                        sliderOptions.Callback(math.floor(sliderOptions.Min + (sliderOptions.Max - sliderOptions.Min) * percent))
                    end
                end)
                uis.InputEnded:Wait()
                conn:Disconnect()
            end)
            return slider
        end

        function tab:AddDropdown(dropdownOptions)
            local dropdown = Create("Instance.new", "TextButton", {Text = "Select", Size = UDim2.new(0, 150, 0, 40), BackgroundColor3 = theme.Background_, Parent = tabFrame})
            local list = Create("Instance.new", "Frame", {Size = UDim2.new(0, 150, 0, #dropdownOptions.Options * 30), BackgroundColor3 = theme.Background_, Visible = false, Parent = tabFrame})

            dropdown.MouseButton1Click:Connect(function() list.Visible = not list.Visible end)
            for _, opt in pairs(dropdownOptions.Options) do
                local btn = Create("Instance.new", "TextButton", {Text = opt, Size = UDim2.new(1, 0, 0, 30), BackgroundColor3 = theme.Background_, Parent = list})
                btn.MouseButton1Click:Connect(function()
                    dropdown.Text = opt
                    list.Visible = false
                    dropdownOptions.Callback(opt)
                end)
            end
            return dropdown
        end

        function tab:AddLabel(labelOptions)
            local lbl = Create("Instance.new", "TextLabel", {Text = labelOptions.Text, Size = UDim2.new(0, 200, 0, 40), BackgroundColor3 = theme.Background_, Parent = tabFrame})
            return lbl
        end

        function tab:AddParagraph(paragraphOptions)
            local para = tab:AddLabel({Text = paragraphOptions.Text})
            para.TextWrapped = true
            para.Size = UDim2.new(0, 200, 0, 100)
            return para
        end

        tabButton.MouseButton1Click:Connect(function()
            for _, t in pairs(window.Tabs) do
                t.Frame.Visible = false
            end
            tabFrame.Visible = true
        end)

        tab.Frame = tabFrame
        table.insert(window.Tabs, tab)
        return tab
    end

    return window
end

return ZenithUI
